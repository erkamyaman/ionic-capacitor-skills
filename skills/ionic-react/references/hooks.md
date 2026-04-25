# React Hooks

Each hook is a thin wrapper around a framework-agnostic util in `utils/` (defined in `ionic-shared`). The hooks that hold **state** (`usePurchases`, `useTheme`, `useOnboarding`, `useNotifications`) keep that state in a module-level store + a `useSyncExternalStore` subscriber, so the same data shows up reactively in every component without prop drilling. This mirrors the **signal-store** pattern used by Angular's services — see [`../../ionic-angular/references/services.md`](../../ionic-angular/references/services.md) for the parallel.

The pattern: a tiny `createStore<T>` helper at the bottom of this file gives every state-holding hook a `getSnapshot` / `subscribe` pair that React renders directly. No Context provider needed; the store is a singleton.

## `hooks/usePurchases.ts`

```typescript
import { useCallback, useSyncExternalStore } from 'react';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';
import { PurchasesPackage } from '@revenuecat/purchases-capacitor';
import { createStore } from './createStore';

const isPremiumStore = createStore(false);

async function refresh() {
  isPremiumStore.set(await isPremiumUser());
}

export function usePurchases() {
  const isPremium = useSyncExternalStore(
    isPremiumStore.subscribe,
    isPremiumStore.get,
  );

  const initialize = useCallback(async () => {
    await initializePurchases();
    await refresh();
  }, []);

  const purchase = useCallback(async (pkg: PurchasesPackage) => {
    const ok = await purchasePackage(pkg);
    if (ok) await refresh();
    return ok;
  }, []);

  const restore = useCallback(async () => {
    const ok = await restorePurchases();
    if (ok) await refresh();
    return ok;
  }, []);

  return {
    isPremium,                  // ← reactive boolean, updates every consumer on purchase/restore
    initialize,
    refresh,                    // exposed in case the caller needs to re-check (e.g. on app resume)
    getOfferings,
    purchase,
    restore,
  };
}
```

Now any component renders `const { isPremium } = usePurchases();` and the value updates the moment a purchase completes — no manual re-fetch.

## `hooks/useTheme.ts`

```typescript
import { useCallback, useSyncExternalStore } from 'react';
import {
  getTheme as readTheme,
  setTheme as persistTheme,
  applyTheme,
  ThemeMode,
} from '../utils/theme';
import { createStore } from './createStore';

const themeStore = createStore<ThemeMode>('system');

export function useTheme() {
  const mode = useSyncExternalStore(themeStore.subscribe, themeStore.get);

  const initialize = useCallback(async () => {
    const saved = await readTheme();
    themeStore.set(saved);
    applyTheme(saved);
  }, []);

  const setMode = useCallback(async (next: ThemeMode) => {
    themeStore.set(next);
    await persistTheme(next);
    applyTheme(next);
  }, []);

  return { mode, initialize, setMode };
}
```

## `hooks/useOnboarding.ts`

```typescript
import { useCallback, useSyncExternalStore } from 'react';
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';
import { createStore } from './createStore';

const completedStore = createStore(false);

export function useOnboarding() {
  const completed = useSyncExternalStore(completedStore.subscribe, completedStore.get);

  const initialize = useCallback(async () => {
    completedStore.set(await isOnboardingCompleted());
  }, []);

  const setCompleted = useCallback(async (value: boolean) => {
    await setOnboardingCompleted(value);
    completedStore.set(value);
  }, []);

  const reset = useCallback(async () => {
    await resetOnboarding();
    completedStore.set(false);
  }, []);

  return { completed, initialize, setCompleted, reset };
}
```

## `hooks/useAds.ts`

`useAds` doesn't own state — banner visibility is just a side-effect call. The premium gate calls the util directly (which re-reads RevenueCat each time):

```typescript
import { useCallback } from 'react';
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

export function useAds() {
  const initialize = useCallback(() => initializeAdMob(), []);
  const showBanner = useCallback(async () => {
    // Always-fresh check via the util. Trades a tiny latency for guaranteed
    // correctness even if usePurchases's store hasn't been refreshed yet.
    // Angular's AdsService, by contrast, reads `purchases.isPremium()`
    // synchronously from the signal store — same correctness, faster path.
    if (await isPremiumUser()) return;
    await showBannerAd();
  }, []);
  const hideBanner = useCallback(() => hideBannerAd(), []);
  return { initialize, showBanner, hideBanner };
}
```

## `hooks/useNotifications.ts`

```typescript
import { useCallback, useSyncExternalStore } from 'react';
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';
import { createStore } from './createStore';

const grantedStore = createStore(false);

export function useNotifications() {
  const permissionGranted = useSyncExternalStore(grantedStore.subscribe, grantedStore.get);

  const requestPermission = useCallback(async () => {
    const granted = await requestNotificationPermission();
    grantedStore.set(granted);
    return granted;
  }, []);

  // Pass-through — the underlying util is module-scoped, no need to wrap.
  return { permissionGranted, requestPermission, addListeners: addNotificationListeners };
}
```

## `hooks/createStore.ts`

```typescript
// Tiny external store — React's useSyncExternalStore reads from this.
export function createStore<T>(initial: T) {
  let value = initial;
  const listeners = new Set<() => void>();

  return {
    get: () => value,
    set: (next: T) => {
      if (Object.is(next, value)) return;
      value = next;
      listeners.forEach((fn) => fn());
    },
    subscribe: (fn: () => void) => {
      listeners.add(fn);
      return () => listeners.delete(fn);
    },
  };
}
```

The store is a closure-captured singleton — every component using the same hook reads the same value.

## Why not Context?

Context works but requires a provider in the tree, prop-equality re-render gotchas, and selector libraries to optimize. `useSyncExternalStore` is the React-blessed primitive for "external mutable state read into React" exactly because Context isn't great at this. For a small fixed set of stores like this scaffold, the explicit pattern is cleaner.

The util sources are documented in:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)
