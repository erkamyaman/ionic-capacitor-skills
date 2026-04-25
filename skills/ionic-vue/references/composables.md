# Vue Composables

Each composable wraps a framework-agnostic util in `utils/` (defined in `ionic-shared`). State-holding composables (`usePurchases`, `useTheme`, `useOnboarding`, `useNotifications`) keep their state in module-level `ref`s — singletons shared across every consumer. This mirrors the **signal-store** pattern used by Angular's services — see [`../../ionic-angular/references/services.md`](../../ionic-angular/references/services.md) for the parallel.

The pattern: declare the `ref` outside the composable body, return it from the composable. Every component that calls the composable reads the same reactive value.

## `composables/usePurchases.ts`

```typescript
import { ref, readonly } from 'vue';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';
import type { PurchasesPackage } from '@revenuecat/purchases-capacitor';

// Module-level singleton — every component shares this ref.
const _isPremium = ref(false);

async function refresh() {
  _isPremium.value = await isPremiumUser();
}

export function usePurchases() {
  async function initialize() {
    await initializePurchases();
    await refresh();
  }

  async function purchase(pkg: PurchasesPackage) {
    const ok = await purchasePackage(pkg);
    if (ok) await refresh();
    return ok;
  }

  async function restore() {
    const ok = await restorePurchases();
    if (ok) await refresh();
    return ok;
  }

  return {
    isPremium: readonly(_isPremium),   // ← reactive boolean; updates every consumer on purchase/restore
    initialize,
    refresh,                            // exposed in case the caller needs to re-check (e.g. on app resume)
    getOfferings,
    purchase,
    restore,
  };
}
```

`readonly(_isPremium)` prevents components from directly mutating the value — they go through `purchase()` / `restore()` / `refresh()`.

## `composables/useTheme.ts`

```typescript
import { ref, readonly } from 'vue';
import {
  getTheme as readTheme,
  setTheme as persistTheme,
  applyTheme,
  type ThemeMode,
} from '../utils/theme';

const _mode = ref<ThemeMode>('system');

export function useTheme() {
  async function initialize() {
    const saved = await readTheme();
    _mode.value = saved;
    applyTheme(saved);
  }

  async function setMode(next: ThemeMode) {
    _mode.value = next;
    await persistTheme(next);
    applyTheme(next);
  }

  return {
    mode: readonly(_mode),
    initialize,
    setMode,
  };
}
```

## `composables/useOnboarding.ts`

```typescript
import { ref, readonly } from 'vue';
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';

const _completed = ref(false);

export function useOnboarding() {
  async function initialize() {
    _completed.value = await isOnboardingCompleted();
  }

  async function setCompleted(value: boolean) {
    await setOnboardingCompleted(value);
    _completed.value = value;
  }

  async function reset() {
    await resetOnboarding();
    _completed.value = false;
  }

  return {
    completed: readonly(_completed),
    initialize,
    setCompleted,
    reset,
  };
}
```

## `composables/useAds.ts`

`useAds` doesn't own state — banner visibility is just a side-effect call. The premium gate calls the util directly (which re-reads RevenueCat each time and is no-op when the SDK hasn't been initialized — see [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)):

```typescript
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

export function useAds() {
  const initialize = () => initializeAdMob();
  const showBanner = async () => {
    // Always-fresh check. Trades a tiny latency for guaranteed correctness
    // even if usePurchases hasn't refreshed yet.
    if (await isPremiumUser()) return;
    await showBannerAd();
  };
  const hideBanner = () => hideBannerAd();
  return { initialize, showBanner, hideBanner };
}
```

## `composables/useNotifications.ts`

```typescript
import { ref, readonly } from 'vue';
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';

const _permissionGranted = ref(false);

export function useNotifications() {
  async function requestPermission() {
    const granted = await requestNotificationPermission();
    _permissionGranted.value = granted;
    return granted;
  }

  return {
    permissionGranted: readonly(_permissionGranted),
    requestPermission,
    addListeners: addNotificationListeners,
  };
}
```

## Why module-level `ref`s instead of `provide` / `inject`?

Both work, but module-level singletons give you the same "single source of truth" with zero boilerplate — no provider component, no inject keys, no risk of forgetting the provider in a test setup. For the small fixed set of stores in this scaffold, the module-singleton pattern is cleaner.

For app-wide stores with more complex needs (devtools, persistence plugins, multiple instances), reach for **Pinia**.

The util sources are documented in:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)
