# Settings Page (Angular)

Required entries:

1. **Language** — switch between supported locales.
2. **Theme** — Light / Dark / System.
3. **Notifications** — toggle that requests permission on enable.
4. **Remove Ads** — navigates to `/paywall`. Hide for premium users.
5. **Reset Onboarding** — clears the flag and routes back to `/onboarding`. Useful for QA / demo.

## `settings/settings.page.html`

```html
<ion-page>
  <ion-header>
    <ion-toolbar>
      <ion-title>{{ 'settings.title' | translate }}</ion-title>
    </ion-toolbar>
  </ion-header>
  <ion-content>
    <ion-list>
      <ion-item>
        <ion-icon name="language" slot="start" aria-hidden="true"></ion-icon>
        <ion-label>{{ 'settings.language' | translate }}</ion-label>
        <ion-select
          [value]="currentLang()"
          (ionChange)="changeLanguage($event)"
          [attr.aria-label]="'settings.language' | translate"
        >
          <ion-select-option value="en">English</ion-select-option>
          <ion-select-option value="tr">Türkçe</ion-select-option>
        </ion-select>
      </ion-item>

      <ion-item>
        <ion-icon name="color-palette" slot="start" aria-hidden="true"></ion-icon>
        <ion-label>{{ 'settings.theme' | translate }}</ion-label>
        <ion-select
          [value]="currentTheme()"
          (ionChange)="changeTheme($event)"
        >
          <ion-select-option value="system">{{ 'settings.system' | translate }}</ion-select-option>
          <ion-select-option value="light">{{ 'settings.light' | translate }}</ion-select-option>
          <ion-select-option value="dark">{{ 'settings.dark' | translate }}</ion-select-option>
        </ion-select>
      </ion-item>

      <ion-item>
        <ion-icon name="notifications" slot="start" aria-hidden="true"></ion-icon>
        <ion-label>{{ 'settings.notifications' | translate }}</ion-label>
        <ion-toggle
          [checked]="notificationsEnabled()"
          (ionChange)="toggleNotifications($event)"
        ></ion-toggle>
      </ion-item>

      @if (!isPremium()) {
        <ion-item button (click)="removeAds()">
          <ion-icon name="star" slot="start" aria-hidden="true"></ion-icon>
          <ion-label>{{ 'settings.removeAds' | translate }}</ion-label>
        </ion-item>
      }

      <ion-item button (click)="resetOnboarding()">
        <ion-icon name="refresh" slot="start" aria-hidden="true"></ion-icon>
        <ion-label>{{ 'settings.resetOnboarding' | translate }}</ion-label>
      </ion-item>
    </ion-list>
  </ion-content>
</ion-page>
```

Note the use of `@if` (built-in control flow — see [control-flow.md](control-flow.md)) and `aria-hidden="true"` on decorative icons (see [accessibility.md](accessibility.md)).

## `settings/settings.page.ts`

```typescript
import {
  Component, ChangeDetectionStrategy, OnInit, inject, signal,
} from '@angular/core';
import { Router } from '@angular/router';
import {
  IonContent, IonHeader, IonTitle, IonToolbar, IonPage,
  IonList, IonItem, IonLabel, IonIcon, IonToggle,
  IonSelect, IonSelectOption,
  IonSelectCustomEvent, IonToggleCustomEvent,
} from '@ionic/angular/standalone';
import { TranslateModule, TranslateService } from '@ngx-translate/core';
import { firstValueFrom } from 'rxjs';
import { LanguageService, type Lang, SUPPORTED_LANGS } from '../services/language.service';
import { ThemeService } from '../services/theme.service';
import { OnboardingService } from '../services/onboarding.service';
import { PurchasesService } from '../services/purchases.service';
import { NotificationsService } from '../services/notifications.service';
import type { ThemeMode } from '../utils/theme';

@Component({
  selector: 'app-settings',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    IonContent, IonHeader, IonTitle, IonToolbar, IonPage,
    IonList, IonItem, IonLabel, IonIcon, IonToggle,
    IonSelect, IonSelectOption,
    TranslateModule,
  ],
  templateUrl: './settings.page.html',
  styleUrls: ['./settings.page.scss'],
})
export class SettingsPage implements OnInit {
  private router = inject(Router);
  private language = inject(LanguageService);
  private theme = inject(ThemeService);
  private onboarding = inject(OnboardingService);
  private purchases = inject(PurchasesService);
  private notifications = inject(NotificationsService);
  private translate = inject(TranslateService);

  // Read-through to service signals (signal-store pattern). These are
  // Signal<T> values — call them with parens in templates: `isPremium()`.
  readonly isPremium = this.purchases.isPremium;
  readonly currentTheme = this.theme.mode;
  readonly currentLang = this.language.lang;
  readonly notificationsEnabled = this.notifications.permissionGranted;

  ngOnInit() {
    // Nothing to hydrate — every reactive value comes from a service signal
    // that's already populated by the APP_INITIALIZER bootstrap (see app-config.md).
  }

  // (ionChange) on <ion-select> emits IonSelectCustomEvent — typed correctly.
  async changeLanguage(event: IonSelectCustomEvent<Lang>) {
    const lang = event.detail.value;
    if (!SUPPORTED_LANGS.includes(lang)) return;       // defensive — ignore unsupported values
    await this.language.setLang(lang);
  }

  changeTheme(event: IonSelectCustomEvent<ThemeMode>) {
    this.theme.setMode(event.detail.value);
  }

  async toggleNotifications(event: IonToggleCustomEvent<{ checked: boolean }>) {
    const enabled = event.detail.checked;
    if (enabled) {
      await this.notifications.requestPermission();
      // The service's permissionGranted signal flips to the OS-confirmed value;
      // if the user denied, the toggle reverts visually on the next render.
    }
    // Toggle-off intentionally does nothing — the OS permission is only revocable
    // via Settings.app. The app may stop *using* push, but the OS still considers
    // permission granted.
  }

  removeAds() {
    this.router.navigateByUrl('/paywall');
  }

  async resetOnboarding() {
    await this.onboarding.reset();
    this.router.navigateByUrl('/onboarding', { replaceUrl: true });
  }
}
```

Notable changes vs. earlier iterations:

- **Typed Ionic events** (`IonSelectCustomEvent<Lang>`, `IonToggleCustomEvent<...>`) replace the manual `(event as CustomEvent<...>).detail` casts. Imports come from `@ionic/angular/standalone`.
- **Language is now a `LanguageService`** (next section). The page no longer writes to `Preferences` directly — that string-key plumbing belongs in a service.
- **`isPremium` / `currentTheme` / `currentLang` / `notificationsEnabled` are read-through aliases to service signals.** No local state shadows. They're `Signal<T>`, so templates use them as `currentTheme()` etc.
- **No `Preferences.set({...})` floating Promise** — language persistence is awaited inside the `LanguageService`.
- **`SUPPORTED_LANGS` validation** prevents an unsupported language from ever reaching `translate.use(...)`, even if Ionic emits an unexpected `value`.

Notes:

- **`ChangeDetectionStrategy.OnPush`** + signals — re-renders only on signal change.
- **`inject()`** for all dependencies, no constructor parameters.
- **Typed `CustomEvent<...>` casts** instead of bare `CustomEvent` — keep TypeScript honest about the `event.detail` shape Ionic emits.
- **`isPremium = this.purchases.isPremium`** — read-through to the service signal; no manual `isPremium()` async call in `ngOnInit`. The service exposes `isPremium` as a `Signal<boolean>` per the signal-store pattern in [services.md](services.md).
- **Built-in `@if`** — no `NgIf` import needed.

