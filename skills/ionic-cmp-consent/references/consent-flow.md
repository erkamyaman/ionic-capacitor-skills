# Consent Flow

The required order on every launch:

```
AdMob.requestConsentInfo()
  → if status === REQUIRED
       → AdMob.showConsentForm()
  → else continue
  → AdMob.initialize()
  → showBannerAd() (only if not premium)
```

## Utility (framework-agnostic)

```typescript
// utils/consent.ts
import { Capacitor } from '@capacitor/core';
import {
  AdMob,
  AdmobConsentStatus,
  AdmobConsentDebugGeography,
} from '@capacitor-community/admob';

export async function gatherConsent(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;

  // Vite (React/Vue): import.meta.env.DEV is the build-mode boolean.
  // Angular CLI: replace with !environment.production from src/environments/environment.ts.
  const isDev = import.meta.env.DEV; // or: !environment.production

  const consentInfo = await AdMob.requestConsentInfo({
    // For testing, pretend we're in the EEA so the form always appears.
    debugGeography: isDev
      ? AdmobConsentDebugGeography.EEA
      : AdmobConsentDebugGeography.DISABLED,
    // Test device hashes — only registered in dev builds.
    testDeviceIdentifiers: isDev ? ['YOUR_TEST_DEVICE_HASH'] : [],
  });

  if (
    consentInfo.isConsentFormAvailable &&
    consentInfo.status === AdmobConsentStatus.REQUIRED
  ) {
    await AdMob.showConsentForm();
  }
}

export async function showPrivacyOptions(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;
  await AdMob.showConsentForm(); // re-renders so the user can change their choice
}

export async function resetConsent(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;
  // QA only — wipes stored consent. Method name is `resetConsentInfo()` in
  // recent @capacitor-community/admob versions; older versions named this
  // differently — check your plugin version's exports if this errors.
  await AdMob.resetConsentInfo();
}
```

## Wire into app startup

Replace the existing `initializeAdMob()` call with:

```typescript
import { gatherConsent } from './utils/consent';
import { initializeAdMob } from './utils/admob';

async function bootstrapAds() {
  await gatherConsent();
  await initializeAdMob();
}
```

Call `bootstrapAds()` from your app's startup hook (Angular `AppComponent.ngOnInit`, React `App.tsx` `useEffect`, Vue `App.vue` `onMounted`).

## Settings page integration

Add a Privacy entry to the existing settings page that calls `showPrivacyOptions()`:

```typescript
import { showPrivacyOptions } from '../utils/consent';

async function openPrivacyOptions() {
  await showPrivacyOptions();
}
```

This is **not optional** — IAB TCF v2 requires the user to be able to change consent at any time.

## Status enum reference

| Status        | Meaning |
|---------------|---------|
| `UNKNOWN`     | UMP hasn't run yet. |
| `NOT_REQUIRED`| User is outside EEA/UK or no message is published. Don't show the form. |
| `REQUIRED`    | User must see the form before personalized ads can serve. |
| `OBTAINED`    | User has already responded. Don't reshow on launch — they can change via Settings. |

Only `REQUIRED` triggers the form on launch.
