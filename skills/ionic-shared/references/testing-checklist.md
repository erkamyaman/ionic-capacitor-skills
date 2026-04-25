# Pre-Release Testing Checklist

Run through this list before any store submission.

## Localization

- [ ] All visible strings come from translation files — no hardcoded copy.
- [ ] UI verified in both `en` and `tr` (or all supported locales).
- [ ] Turkish strings use correct characters (`ı İ ü ö ç ş ğ`).
- [ ] Long-string layouts don't break (German / Turkish translations are typically longer than English).

## Theme

- [ ] Light mode rendered correctly.
- [ ] Dark mode rendered correctly.
- [ ] System mode follows the OS preference and reacts to changes when the app is open.

## Onboarding & paywall

- [ ] First-launch flow: onboarding → paywall → tabs.
- [ ] "Reset Onboarding" in Settings restarts the flow.
- [ ] Onboarding guard prevents reaching `/tabs` while onboarding is incomplete.
- [ ] Paywall offers both packages (weekly / yearly) and the yearly badge is visible.
- [ ] Subscribe → tabs (with entitlement active).
- [ ] Continue with ads → tabs (no entitlement).
- [ ] Restore Purchases works for an account that previously purchased.

## RevenueCat / IAP

- [ ] Sandbox / closed-testing purchase succeeds.
- [ ] `isPremiumUser()` returns `true` after purchase.
- [ ] AdMob banner is hidden for premium users.
- [ ] Cancelling a purchase does not show an error.

## AdMob

- [ ] Banner shows on tabs for non-premium users.
- [ ] Banner hides on paywall and onboarding.
- [ ] Test IDs in dev builds, production IDs in release builds.

## Notifications

- [ ] Permission prompt fires from the Settings toggle, not on launch.
- [ ] Token logged successfully in `registration` listener.
- [ ] Test push from FCM/APNs renders correctly in foreground and background.

## Platform coverage

- [ ] `ionic serve` (browser) — happy path works.
- [ ] iOS simulator.
- [ ] iOS real device (App Tracking Transparency, IAP sandbox).
- [ ] Android emulator.
- [ ] Android real device (closed-testing IAP, FCM push).

## Misc

- [ ] Offline behavior: app doesn't crash when network is unavailable.
- [ ] Multiple screen sizes (small phone, large phone, tablet).
- [ ] Correct `appId` and `appName` in `capacitor.config.ts`.
- [ ] No `console.error` output in a clean session.
