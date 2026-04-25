---
name: ionic-shared
description: Framework-agnostic Capacitor concerns shared by Ionic Angular, React, and Vue apps — AdMob, RevenueCat, push notifications, @capacitor/preferences storage, theme switching, localization content, and App/Play store submission notes. Referenced by ionic-angular, ionic-react, and ionic-vue.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Ionic Shared (Capacitor) Guidelines

Cross-framework guidance for Ionic Capacitor apps. The framework skills (`ionic-angular`, `ionic-react`, `ionic-vue`) link here for native plugin and content topics so the same code is not repeated three times.

## When to consult these references

- **Capacitor config / native platforms**: [capacitor-config.md](references/capacitor-config.md)
- **Environments & API keys** (env vars per framework, what's safe to ship): [environments-and-keys.md](references/environments-and-keys.md)
- **Persistent storage** (`@capacitor/preferences`): [storage.md](references/storage.md)
- **Theme switching** (Light / Dark / System): [theming.md](references/theming.md)
- **AdMob banner ads**: [admob.md](references/admob.md)
- **RevenueCat purchases / paywall logic**: [revenuecat.md](references/revenuecat.md)
- **Push notifications**: [push-notifications.md](references/push-notifications.md)
- **Translation strings (TR / EN)**: [localization-content.md](references/localization-content.md)
- **App Store / Play Store submission notes**: [app-store-notes.md](references/app-store-notes.md)
- **Pre-release testing checklist**: [testing-checklist.md](references/testing-checklist.md)

## Hard rules (apply to all frameworks)

- ✅ **Use the latest stable versions** of Ionic Framework, Capacitor, and the framework (Angular / React / Vue). When scaffolding a new project, do not pin to older majors — let `npm install <pkg>` resolve the latest. Verify against `npm view <pkg> version` if uncertain. Do not write specific version numbers into `SKILL.md` examples — they go stale.
- ✅ **Read API keys and ad unit IDs from environment variables**, never hardcoded in source. The mobile bundle is not a secret container — only **publishable** keys (RevenueCat SDK key, AdMob unit ID, Firebase web config, Sentry DSN) belong in the app. **Secret** keys (Stripe `sk_…`, OpenAI / Anthropic, RevenueCat REST secret) MUST live behind a backend. See [environments-and-keys.md](references/environments-and-keys.md).
- ✅ **Tie `isTesting` flags to the build mode**, not a hardcoded `true`. Shipping `initializeForTesting: true` or `isTesting: true` to production can get an AdMob account banned.
- ❌ NEVER use `localStorage` directly — use `@capacitor/preferences`.
- ❌ NEVER use `@ionic/storage` — use `@capacitor/preferences`.
- ❌ NEVER use `cordova-plugin-*` — use the Capacitor equivalent.
- ❌ NEVER use `ngx-admob-free` or other deprecated ad libraries — use `@capacitor-community/admob`.
- ❌ NEVER call Capacitor plugin methods synchronously — always `await`.
- ✅ ALWAYS guard native-only code with `Capacitor.isNativePlatform()`.
- ✅ ALWAYS run `npx cap sync` after installing or updating any Capacitor plugin.

## Required shared libraries

```bash
npm install \
  @capacitor/app \
  @capacitor/preferences \
  @capacitor/push-notifications \
  @capacitor/splash-screen \
  @capacitor/status-bar \
  @revenuecat/purchases-capacitor \
  @capacitor-community/admob \
  swiper
```

`@capacitor/app` is needed by `ionic-deep-links` (and any skill that listens to `appUrlOpen` / `appStateChange`), so it's part of the shared baseline.

These are installed in addition to the framework-specific i18n library (see each framework's `SKILL.md`).
