# App Store & Play Store Submission Notes

## Build pipeline before submission

```bash
npm run build      # or: ionic build
npx cap sync       # copy web assets + plugins into native projects
npx cap open ios   # Xcode → Archive → Distribute
npx cap open android   # Android Studio → Build → Generate Signed Bundle
```

`cap sync` MUST run after every web build — skipping it ships stale assets to native.

## iOS

- **App Tracking Transparency**: required for personalized ads. Add `NSUserTrackingUsageDescription` to `Info.plist` and request via the ATT plugin **after** onboarding (never on first launch — Apple rejects).
- **Restore Purchases**: the paywall must include a Restore Purchases button. App Review checks for this explicitly when In-App Purchase is enabled.
- **Permissions**: each Capacitor plugin you install adds entries to `Info.plist`. Verify they have human-readable usage descriptions (App Review rejects placeholder text).
- **Target SDK**: Apple periodically raises the minimum Xcode version — check the current requirement before submitting.
- **Sandbox testing**: configure a Sandbox tester in App Store Connect → Users and Access → Sandbox.

## Android

- **Permissions**: declared in `android/app/src/main/AndroidManifest.xml` (added automatically by Capacitor plugins).
- **Target SDK**: Play Console enforces a minimum target SDK each year — check Play Console's Pre-launch report for warnings.
- **Closed testing**: required to test in-app purchases. Set up a closed testing track and add license tester accounts in Play Console → Setup → License testing.
- **`google-services.json`**: required for FCM push notifications. Place in `android/app/`.

## RevenueCat checklist (both stores)

- API keys configured per platform.
- Products created in App Store Connect and Play Console with the same product IDs as configured in RevenueCat.
- Entitlements mapped in RevenueCat dashboard.
- Restore Purchases button verified working.

## Common rejection causes

- Asking for push notification or ATT permission immediately on first launch.
- Missing Restore Purchases button when IAP is present.
- Using non-test ad IDs in test builds (Google can ban the AdMob account).
- `Info.plist` permission strings that look like placeholder text.
- Targeting an outdated SDK / API level.
