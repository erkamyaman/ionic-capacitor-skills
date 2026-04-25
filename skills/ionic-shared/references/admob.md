# AdMob (`@capacitor-community/admob`)

Banner ads, hidden when the user is premium (RevenueCat entitlement active).

## Utility (framework-agnostic)

```typescript
// utils/admob.ts
import {
  AdMob,
  BannerAdOptions,
  BannerAdSize,
  BannerAdPosition,
} from '@capacitor-community/admob';
import { Capacitor } from '@capacitor/core';

let initialized = false;

export async function initializeAdMob(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;

  await AdMob.initialize({
    initializeForTesting: true, // set to false in production
  });
  initialized = true;
}

export async function showBannerAd(): Promise<void> {
  if (!initialized) return;

  const options: BannerAdOptions = {
    adId: 'ca-app-pub-3940256099942544/6300978111', // test ID — replace in production
    adSize: BannerAdSize.ADAPTIVE_BANNER,
    position: BannerAdPosition.BOTTOM_CENTER,
    isTesting: true, // set to false in production
  };

  await AdMob.showBanner(options);
}

export async function hideBannerAd(): Promise<void> {
  if (!initialized) return;
  await AdMob.hideBanner();
}
```

## Test Ad IDs

Use Google's test IDs during development — using a real ad unit ID for testing can get the AdMob account banned.

| Format | Test ID |
|--------|---------|
| Banner | `ca-app-pub-3940256099942544/6300978111` |

> **Don't hardcode the ad unit ID or the testing flag.** Read both from environment variables tied to the build mode. The literal `'ca-app-pub-3940256099942544/6300978111'` and `isTesting: true` shown above are placeholders for a development env file — the production env file points at the real ad unit ID with `isTesting: false`. See [environments-and-keys.md](environments-and-keys.md).

```typescript
const ADMOB_TESTING = import.meta.env.VITE_ADMOB_TESTING === 'true';     // Vite
// const ADMOB_TESTING = !environment.production;                         // Angular CLI
const BANNER_ID = import.meta.env.VITE_ADMOB_BANNER_ID;
```

Then use those constants instead of the literals in the snippet above.

## When to show / hide

- Initialize on app startup.
- Show the banner when entering the tabs layout, **only** if `isPremiumUser()` returns `false` (see [revenuecat.md](revenuecat.md)).
- Hide on tabs unmount / page leave so it doesn't bleed into modals or full-screen pages (paywall, onboarding).

## iOS App Tracking Transparency

Personalized ads on iOS 14+ require requesting ATT permission. Without it, ads are non-personalized (lower revenue). If the project plans to monetize via personalized ads, request ATT after onboarding completes — see App Store notes.
