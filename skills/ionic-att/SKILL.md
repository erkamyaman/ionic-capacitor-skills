---
name: ionic-att
description: Request iOS App Tracking Transparency (ATT) permission so AdMob can serve personalized ads on iOS 14+ without App Review rejecting the build. Trigger when adding ATT, IDFA permission, or "iOS personalized ads" to an Ionic Capacitor app.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# iOS App Tracking Transparency (ATT)

iOS 14+ requires apps that track users across other companies' apps and websites — including AdMob personalized ads — to ask for permission via the system ATT dialog. Without it: ads degrade to non-personalized, eCPM drops dramatically, and Apple rejects the build during review.

## When to consult

- **Info.plist setup + plugin install**: [setup.md](references/setup.md)
- **When and how to prompt** (timing matters): [request-flow.md](references/request-flow.md)
- **Per-framework integration**: [framework-snippets.md](references/framework-snippets.md)

## Hard rules

- ❌ **NEVER prompt on first launch.** Apple rejects this. The ATT dialog must come after the user has experienced enough of the app to understand why permission is being requested.
- ✅ Prompt **after onboarding completes** and **before** AdMob's first impression.
- ✅ The `NSUserTrackingUsageDescription` string in `Info.plist` MUST clearly state how the data is used. Generic placeholders ("We need this to show ads") get rejected.
- ✅ Android is not affected — the ATT plugin no-ops on Android.
- ✅ ATT pairs with [`ionic-cmp-consent`](../ionic-cmp-consent/SKILL.md) — they cover different jurisdictions (Apple vs EU), and both are required for full ad revenue.

## Library

```bash
npm install @capacitor-community/app-tracking-transparency
npx cap sync
```
