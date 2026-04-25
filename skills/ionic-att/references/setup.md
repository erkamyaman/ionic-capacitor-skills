# ATT Setup

## Plugin install

```bash
npm install @capacitor-community/app-tracking-transparency
npx cap sync
```

## `Info.plist` — required

Open `ios/App/App/Info.plist` (or via `npx cap open ios` → file inspector) and add:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>We use this to show ads relevant to your interests and measure their performance.</string>
```

The string is shown verbatim in the system dialog. **Apple rejects placeholder text or anything that looks generated.** Be specific to your app — if it's a fitness app, mention fitness; if it's a finance app, explain what categories of ads.

Examples Apple has accepted:

- ✅ "Your data will be used to deliver personalized recipe and meal-kit ads."
- ✅ "We use this to measure ad performance and show you relevant offers from our partners."

Examples that have been rejected:

- ❌ "We use your data."
- ❌ "Required for ad personalization."
- ❌ "[NSUserTrackingUsageDescription]" (placeholder text not replaced)

## SKAdNetwork (also required)

For accurate attribution of installs from non-personalized ads, add SKAdNetwork IDs to `Info.plist`. AdMob publishes the canonical list at <https://developers.google.com/admob/ios/ios14> — paste the entire `SKAdNetworkItems` block they provide.

Without SKAdNetwork, your ad attribution is broken even before ATT.

## Verify

```bash
npx cap sync
npx cap open ios
# In Xcode: Build → Show Build Folder → look for Info.plist; check the key is present
```
