# Angular Services

Each service is a thin `@Injectable({ providedIn: 'root' })` wrapper around the framework-agnostic `utils/` functions defined in `ionic-shared`. Services exist so components can inject the surface they need rather than calling utility functions directly.

## `services/onboarding.service.ts`

```typescript
import { Injectable } from '@angular/core';
import {
  isOnboardingCompleted,
  setOnboardingCompleted,
  resetOnboarding,
} from '../utils/onboarding';

@Injectable({ providedIn: 'root' })
export class OnboardingService {
  isCompleted = isOnboardingCompleted;
  setCompleted = setOnboardingCompleted;
  reset = resetOnboarding;
}
```

## `services/theme.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { getTheme, setTheme, applyTheme, ThemeMode } from '../utils/theme';

@Injectable({ providedIn: 'root' })
export class ThemeService {
  async initialize(): Promise<void> {
    const theme = await getTheme();
    applyTheme(theme);
  }

  getTheme = getTheme;
  setTheme = setTheme;
}
```

## `services/ads.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { initializeAdMob, showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

@Injectable({ providedIn: 'root' })
export class AdsService {
  async initialize(): Promise<void> {
    await initializeAdMob();
  }

  async showBanner(): Promise<void> {
    if (await isPremiumUser()) return;
    await showBannerAd();
  }

  async hideBanner(): Promise<void> {
    await hideBannerAd();
  }
}
```

## `services/purchases.service.ts`

```typescript
import { Injectable } from '@angular/core';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';

@Injectable({ providedIn: 'root' })
export class PurchasesService {
  initialize = initializePurchases;
  isPremium = isPremiumUser;
  getOfferings = getOfferings;
  purchase = purchasePackage;
  restorePurchases = restorePurchases;
}
```

## `services/notifications.service.ts`

```typescript
import { Injectable } from '@angular/core';
import {
  requestNotificationPermission,
  addNotificationListeners,
} from '../utils/notifications';

@Injectable({ providedIn: 'root' })
export class NotificationsService {
  requestPermission = requestNotificationPermission;
  addListeners = addNotificationListeners;
}
```

The `utils/*.ts` source for each utility lives in:

- [`../../ionic-shared/references/storage.md`](../../ionic-shared/references/storage.md) (onboarding util)
- [`../../ionic-shared/references/theming.md`](../../ionic-shared/references/theming.md)
- [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md)
- [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md)
- [`../../ionic-shared/references/push-notifications.md`](../../ionic-shared/references/push-notifications.md)
