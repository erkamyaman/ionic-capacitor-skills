# React Hooks

Each hook is a thin wrapper around a framework-agnostic util in `utils/`. The utils themselves come from `ionic-shared`.

## `hooks/useOnboarding.ts`

```typescript
import { useCallback } from 'react';
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';

export function useOnboarding() {
  const isCompleted = useCallback(() => isOnboardingCompleted(), []);
  const setCompleted = useCallback((v: boolean) => setOnboardingCompleted(v), []);
  const reset = useCallback(() => resetOnboarding(), []);
  return { isCompleted, setCompleted, reset };
}
```

## `hooks/useTheme.ts`

```typescript
import { useCallback } from 'react';
import { getTheme, setTheme, applyTheme, ThemeMode } from '../utils/theme';

export function useTheme() {
  const initialize = useCallback(async () => {
    const theme = await getTheme();
    applyTheme(theme);
  }, []);

  return {
    initialize,
    getTheme: useCallback(() => getTheme(), []),
    setTheme: useCallback((mode: ThemeMode) => setTheme(mode), []),
  };
}
```

## `hooks/useAds.ts`

```typescript
import { useCallback } from 'react';
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

export function useAds() {
  const initialize = useCallback(() => initializeAdMob(), []);
  const showBanner = useCallback(async () => {
    if (await isPremiumUser()) return;
    await showBannerAd();
  }, []);
  const hideBanner = useCallback(() => hideBannerAd(), []);
  return { initialize, showBanner, hideBanner };
}
```

## `hooks/usePurchases.ts`

```typescript
import { useCallback } from 'react';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';
import { PurchasesPackage } from '@revenuecat/purchases-capacitor';

export function usePurchases() {
  return {
    initialize: useCallback(() => initializePurchases(), []),
    isPremium: useCallback(() => isPremiumUser(), []),
    getOfferings: useCallback(() => getOfferings(), []),
    purchase: useCallback((pkg: PurchasesPackage) => purchasePackage(pkg), []),
    restorePurchases: useCallback(() => restorePurchases(), []),
  };
}
```

## `hooks/useNotifications.ts`

```typescript
import { useCallback } from 'react';
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';

export function useNotifications() {
  return {
    requestPermission: useCallback(() => requestNotificationPermission(), []),
    addListeners: useCallback(() => addNotificationListeners(), []),
  };
}
```

The util sources are documented in:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)
