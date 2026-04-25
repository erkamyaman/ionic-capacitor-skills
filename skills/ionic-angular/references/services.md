# Angular Services

Each service is an `@Injectable({ providedIn: 'root' })` wrapper around the framework-agnostic `utils/` functions defined in `ionic-shared`. Services follow the **signal-store pattern**: state is held in private `WritableSignal`s exposed as readonly `Signal`s, and methods are the only way to mutate state. Components bind to those signals directly so templates re-render reactively without `async` pipes or manual change detection.

For the underlying signal API (`signal`, `computed`, `effect`), see [signals.md](signals.md).

## `services/onboarding.service.ts`

```typescript
import { Injectable, signal, Signal } from '@angular/core';
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';

@Injectable({ providedIn: 'root' })
export class OnboardingService {
  private readonly _completed = signal(false);
  readonly completed: Signal<boolean> = this._completed.asReadonly();

  async initialize(): Promise<void> {
    this._completed.set(await isOnboardingCompleted());
  }

  async setCompleted(value: boolean): Promise<void> {
    await setOnboardingCompleted(value);
    this._completed.set(value);
  }

  async reset(): Promise<void> {
    await resetOnboarding();
    this._completed.set(false);
  }
}
```

## `services/theme.service.ts`

```typescript
import { Injectable, signal, Signal } from '@angular/core';
import {
  getTheme, setTheme as persistTheme, applyTheme, ThemeMode,
} from '../utils/theme';

@Injectable({ providedIn: 'root' })
export class ThemeService {
  private readonly _mode = signal<ThemeMode>('system');
  readonly mode: Signal<ThemeMode> = this._mode.asReadonly();

  async initialize(): Promise<void> {
    const saved = await getTheme();
    this._mode.set(saved);
    applyTheme(saved);
  }

  async setMode(mode: ThemeMode): Promise<void> {
    this._mode.set(mode);
    // persistTheme (= setTheme from utils/theme.ts) calls applyTheme() internally — see ../../ionic-shared/references/theming.md
    await persistTheme(mode);
  }
}
```

> **No `effect()` here on purpose.** A `constructor` `effect(() => applyTheme(this._mode()))` would fire once with the default `'system'` value before `initialize()` reads the saved mode — causing a brief light/dark flash on cold start. Calling `applyTheme()` explicitly from `initialize()` and letting `persistTheme()` apply it inside `setMode()` keeps the signal-to-DOM bridge predictable and one-shot per write.

For the version that uses `effect()` as a teaching example of pushing signal state to a non-Angular system, see [signals.md](signals.md).

## `services/ads.service.ts`

```typescript
import { Injectable, inject } from '@angular/core';
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { PurchasesService } from './purchases.service';

@Injectable({ providedIn: 'root' })
export class AdsService {
  private purchases = inject(PurchasesService);

  async initialize(): Promise<void> {
    await initializeAdMob();
  }

  async showBanner(): Promise<void> {
    if (this.purchases.isPremium()) return;       // signal read — no await
    await showBannerAd();
  }

  async hideBanner(): Promise<void> {
    await hideBannerAd();
  }
}
```

The premium check is a synchronous signal read (`this.purchases.isPremium()`), no longer an `async` boundary. That's only correct because the `PurchasesService` keeps the entitlement state in a signal and refreshes it via `initialize()` / `refresh()` — see below.

## `services/purchases.service.ts`

```typescript
import { Injectable, signal, Signal, computed } from '@angular/core';
import { Capacitor } from '@capacitor/core';
import { Purchases, PurchasesPackage } from '@revenuecat/purchases-capacitor';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';

@Injectable({ providedIn: 'root' })
export class PurchasesService {
  private readonly _isPremium = signal(false);
  readonly isPremium: Signal<boolean> = this._isPremium.asReadonly();

  readonly tier = computed(() => this._isPremium() ? 'premium' : 'free');

  async initialize(): Promise<void> {
    await initializePurchases();
    await this.refresh();

    // Subscribe to push updates from RevenueCat — fires when an entitlement
    // is granted, expires, or is revoked while the app is open. Without this,
    // a subscription that lapses mid-session keeps the UI in "premium mode"
    // until next launch.
    if (Capacitor.isNativePlatform()) {
      await Purchases.addCustomerInfoUpdateListener((info) => {
        const active = Object.keys(info.entitlements.active).length > 0;
        this._isPremium.set(active);
      });
    }
  }

  async refresh(): Promise<void> {
    this._isPremium.set(await isPremiumUser());
  }

  /** Loads the current offering's available packages. */
  getOfferings(): Promise<PurchasesPackage[]> {
    return getOfferings();
  }

  async purchase(pkg: PurchasesPackage): Promise<boolean> {
    const ok = await purchasePackage(pkg);
    if (ok) await this.refresh();
    return ok;
  }

  async restore(): Promise<boolean> {
    const ok = await restorePurchases();
    if (ok) await this.refresh();
    return ok;
  }
}
```

## `services/language.service.ts`

Owns the active language signal + persistence. Pages read `language.lang()` and call `language.setLang('tr')` — the page never touches `@capacitor/preferences` or `TranslateService` directly. This keeps the `'language'` Preferences key string in one place and means swapping i18n libraries (e.g., to `@angular/localize`) only touches this file.

```typescript
import { Injectable, signal, Signal, inject } from '@angular/core';
import { Preferences } from '@capacitor/preferences';
import { TranslateService } from '@ngx-translate/core';
import { firstValueFrom } from 'rxjs';

export const SUPPORTED_LANGS = ['en', 'tr'] as const;
export type Lang = typeof SUPPORTED_LANGS[number];

const KEY = 'language';

function isLang(v: string | null | undefined): v is Lang {
  return v != null && (SUPPORTED_LANGS as readonly string[]).includes(v);
}

@Injectable({ providedIn: 'root' })
export class LanguageService {
  private translate = inject(TranslateService);

  private readonly _lang = signal<Lang>('en');
  readonly lang: Signal<Lang> = this._lang.asReadonly();

  /** Called from APP_INITIALIZER. Resolves saved → browser → 'en'. */
  async initialize(): Promise<void> {
    const { value } = await Preferences.get({ key: KEY });
    const saved = isLang(value) ? value : null;

    const browser = navigator.language.split('-')[0];
    const fromBrowser = isLang(browser) ? browser : null;

    const next: Lang = saved ?? fromBrowser ?? 'en';
    this._lang.set(next);
    this.translate.setDefaultLang('en');
    await firstValueFrom(this.translate.use(next));   // wait for JSON to load before first paint
  }

  async setLang(next: Lang): Promise<void> {
    if (this._lang() === next) return;
    this._lang.set(next);
    await firstValueFrom(this.translate.use(next));
    await Preferences.set({ key: KEY, value: next });
  }
}
```

If you adopt this, drop the inline `translate.setDefaultLang` / `translate.use` block from `app.config.ts`'s `APP_INITIALIZER` and call `language.initialize()` instead — same effect, but the language plumbing is now owned by the service.

## `services/notifications.service.ts`

```typescript
import { Injectable, signal, Signal } from '@angular/core';
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';

@Injectable({ providedIn: 'root' })
export class NotificationsService {
  private readonly _permissionGranted = signal(false);
  readonly permissionGranted: Signal<boolean> = this._permissionGranted.asReadonly();

  async requestPermission(): Promise<boolean> {
    const granted = await requestNotificationPermission();
    this._permissionGranted.set(granted);
    return granted;
  }

  /** Wires push-notification listeners (registration, errors, received, action). */
  addListeners(): Promise<void> {
    return addNotificationListeners();
  }
}
```

## Bootstrap

Most services have an `initialize()` to hydrate state from storage / native APIs. **These run inside an `APP_INITIALIZER` provider** (see [app-config.md](app-config.md)), not in `AppComponent.ngOnInit`. The reason: `APP_INITIALIZER` blocks Angular bootstrap until its Promise resolves, which means routing — and therefore guards like [`onboarding-guard.md`](onboarding-guard.md) — genuinely cannot fire against unhydrated signals. `ngOnInit`-based bootstrap loses this race.

Order matters:

```
translate.use(lang)            ← await, so first paint shows the right copy
  ↓
theme.initialize() + onboarding.initialize()  ← parallel, no deps between them
  ↓
purchases.initialize()         ← must complete before ads reads premium status
  ↓
ads.initialize()
```

> **Cross-framework note**: React (`useSyncExternalStore` hooks) and Vue (module-level `ref` composables) don't have an `APP_INITIALIZER` equivalent — their bootstrap runs in `App.tsx` / `App.vue` `useEffect`/`onMounted`, gated by a `ready` flag that hides the router outlet until init resolves. Their onboarding guard reads `@capacitor/preferences` directly on each navigation, sidestepping the bootstrap-order question entirely. Angular's signal-store guard is faster (sync read) but couples correctness to bootstrap discipline.

## A word on coupling

`AdsService` injects `PurchasesService` to read `isPremium()` — convenient but a coupling smell at scale. As the app grows, "should I show ads" becomes a cross-cutting policy that other features (e.g., a "Pro" content gate) also care about. At ~10+ services it's worth extracting an `EntitlementsService` (or letting individual pages query `PurchasesService` directly) instead of pulling premium logic through `AdsService`.

For this scaffold's size (5 services), the inline injection is fine — but flag it as a refactor candidate the first time a second consumer of premium state appears.

## What this gives you

- **Templates re-render reactively.** `@if (purchases.isPremium()) { ... }` works without `async` pipes.
- **No manual `OnInit` plumbing in pages.** `SettingsPage` reads `isPremium` directly from the service signal instead of calling `await isPremium()` in `ngOnInit` and storing it in a local field.
- **Single source of truth.** The Settings page's "Remove Ads" button hides the moment a purchase succeeds — `PurchasesService.purchase()` calls `refresh()` which updates the signal, and every component bound to it updates automatically.

The util-source files (`utils/onboarding.ts`, `utils/theme.ts`, etc.) come from `ionic-shared`:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)
