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

// Tied to the build mode — see ../environments-and-keys.md for why this is a boolean
// (not a string env var) and how to wire it for Angular CLI.
const ADMOB_TESTING = import.meta.env.DEV;
const BANNER_ID =
  import.meta.env.VITE_ADMOB_BANNER_ID
  ?? 'ca-app-pub-3940256099942544/6300978111'; // Google's banner test ID — fallback for first-run

export async function initializeAdMob(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;

  await AdMob.initialize({
    initializeForTesting: ADMOB_TESTING,
  });
  initialized = true;
}

export async function showBannerAd(): Promise<void> {
  if (!initialized) return;

  const options: BannerAdOptions = {
    adId: BANNER_ID,
    adSize: BannerAdSize.ADAPTIVE_BANNER,
    position: BannerAdPosition.BOTTOM_CENTER,
    isTesting: ADMOB_TESTING,
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

| Format       | Test ID |
|--------------|---------|
| Banner       | `ca-app-pub-3940256099942544/6300978111` |
| Interstitial | `ca-app-pub-3940256099942544/1033173712` |
| Rewarded     | `ca-app-pub-3940256099942544/5224354917` |

> **Don't hardcode the ad unit ID or the testing flag.** Read both from environment variables tied to the build mode. The literal `'ca-app-pub-3940256099942544/6300978111'` and `isTesting: true` shown above are placeholders for a development env file — the production env file points at the real ad unit ID with `isTesting: false`. See [environments-and-keys.md](environments-and-keys.md).

```typescript
const ADMOB_TESTING = import.meta.env.DEV;                  // Vite (React / Vue) — boolean, typo-proof
// const ADMOB_TESTING = !environment.production;            // Angular CLI
const BANNER_ID = import.meta.env.VITE_ADMOB_BANNER_ID;     // Vite — string env var is fine for the ID
```

Then use those constants instead of the literals in the snippet above.

## When to show / hide

- Initialize on app startup.
- Show the banner when entering the tabs layout, **only** if `isPremiumUser()` returns `false` (see [revenuecat.md](revenuecat.md)).
- Hide on tabs unmount / page leave so it doesn't bleed into modals or full-screen pages (paywall, onboarding).

## Compliance — pair with ATT and UMP

Two compliance prerequisites for serving personalized AdMob ads. Both have dedicated skills; this banner-ad skill assumes their flows have already run.

| Concern | When required | Skill |
|---------|---------------|-------|
| iOS App Tracking Transparency | iOS 14+ when serving personalized ads | [`../../ionic-att/SKILL.md`](../../ionic-att/SKILL.md) |
| Google UMP / GDPR consent     | EU / UK / Switzerland users           | [`../../ionic-cmp-consent/SKILL.md`](../../ionic-cmp-consent/SKILL.md) |

The required full sequence at app start:

```
(iOS only) AppTrackingTransparency.requestPermission()
  ↓
gatherConsent() (UMP)
  ↓
AdMob.initialize()
  ↓
showBannerAd()  (only if not premium)
```

Without ATT on iOS: ads serve non-personalized (lower revenue) and Apple may reject the build during review. Without UMP in the EU: AdMob may eventually disable serving for non-compliant apps. See the app-store notes in [app-store-notes.md](app-store-notes.md).
