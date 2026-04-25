# Vue Composables

Each composable wraps the framework-agnostic `utils/` functions. The utils themselves come from `ionic-shared`.

## `composables/useOnboarding.ts`

```typescript
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';

export function useOnboarding() {
  return {
    isCompleted: isOnboardingCompleted,
    setCompleted: setOnboardingCompleted,
    reset: resetOnboarding,
  };
}
```

## `composables/useTheme.ts`

```typescript
import { getTheme, setTheme, applyTheme, ThemeMode } from '../utils/theme';

export function useTheme() {
  const initialize = async () => {
    const theme = await getTheme();
    applyTheme(theme);
  };

  return { initialize, getTheme, setTheme };
}
```

## `composables/useAds.ts`

```typescript
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

export function useAds() {
  const initialize = () => initializeAdMob();
  const showBanner = async () => {
    if (await isPremiumUser()) return;
    await showBannerAd();
  };
  const hideBanner = () => hideBannerAd();
  return { initialize, showBanner, hideBanner };
}
```

## `composables/usePurchases.ts`

```typescript
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';

export function usePurchases() {
  return {
    initialize: initializePurchases,
    isPremium: isPremiumUser,
    getOfferings,
    purchase: purchasePackage,
    restorePurchases,
  };
}
```

## `composables/useNotifications.ts`

```typescript
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';

export function useNotifications() {
  return {
    requestPermission: requestNotificationPermission,
    addListeners: addNotificationListeners,
  };
}
```

The util sources are documented in:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)

## Why no `useCallback`-equivalent?

Unlike React, Vue's reactivity doesn't re-create function identity on every render — composables can return the bare functions directly. Wrapping in `computed()` or `ref()` is unnecessary unless you need reactive state.
