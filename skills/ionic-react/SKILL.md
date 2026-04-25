---
name: ionic-react
description: Build a production-ready Ionic Capacitor mobile app with React functional components, hooks for cross-cutting concerns, react-router via @ionic/react-router, and react-i18next for i18n. Trigger when the user wants to create an Ionic React app, scaffold pages (onboarding, paywall, tabs, settings), or wire RevenueCat / AdMob / push notifications into a React Ionic project.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Ionic + React App Guidelines

> **This is a SKILL file, NOT a project.** Never run `npm install` here. When creating a new project, ask for the project path or scaffold in a separate directory (e.g. `~/Projects/<app>`).

## When to consult these references

Project setup:

- **Create a new React Ionic project**: [new-app.md](references/new-app.md)
- **Project structure**: [project-structure.md](references/project-structure.md)
- **App configuration** (`main.tsx`, `App.tsx`, `i18n/index.ts`): [app-config.md](references/app-config.md)
- **Routing** (`IonReactRouter`, tab routes): [routing.md](references/routing.md)

Pages and navigation:

- **Onboarding guard** (component wrapper): [onboarding-guard.md](references/onboarding-guard.md)
- **Onboarding page** (video background + Swiper): [onboarding-page.md](references/onboarding-page.md)
- **Paywall page** (RevenueCat): [paywall-page.md](references/paywall-page.md)
- **Tabs layout** (with banner ad lifecycle): [tabs-navigation.md](references/tabs-navigation.md)
- **Settings page**: [settings-page.md](references/settings-page.md)

Cross-cutting:

- **Hooks** (Theme / Onboarding / Ads / Purchases / Notifications): [hooks.md](references/hooks.md)
- **i18n with `react-i18next`**: [i18n-react-i18next.md](references/i18n-react-i18next.md)
- **Best practices** (functional + `IonPage`): [best-practices.md](references/best-practices.md)
- **Commands** (build / sync / open): [commands.md](references/commands.md)

Native plugin topics live in `../ionic-shared/references/`:

- [admob.md](../ionic-shared/references/admob.md)
- [revenuecat.md](../ionic-shared/references/revenuecat.md)
- [push-notifications.md](../ionic-shared/references/push-notifications.md)
- [storage.md](../ionic-shared/references/storage.md)
- [theming.md](../ionic-shared/references/theming.md)
- [localization-content.md](../ionic-shared/references/localization-content.md)

## Required pages (always create)

- **Onboarding** — swipe-based, fullscreen background video + gradient overlay.
- **Paywall** — RevenueCat (weekly / yearly), shown immediately after onboarding.
- **Settings** — language, theme, notifications, remove ads, reset onboarding.

## Required navigation

Use `IonTabs` + `IonTabBar`. Never build a custom tab bar or pull in a third-party tab library.

## Required libraries

```bash
npm install \
  @capacitor/preferences @capacitor/push-notifications \
  @capacitor/splash-screen @capacitor/status-bar \
  @revenuecat/purchases-capacitor @capacitor-community/admob \
  react-i18next i18next i18next-http-backend \
  swiper
```

## Hard rules (React-specific)

- ❌ Class components — use **functional components with hooks**.
- ❌ Direct DOM manipulation — use refs / state.
- ❌ Importing from `@ionic/angular` or `@ionic/vue` — only `@ionic/react`.
- ❌ `any` type.
- ✅ Wrap every page in `<IonPage>` so Ionic's transitions and lifecycle work.

## Before reporting done

```bash
npm install
npm run build
npx cap sync
```

The build must complete without errors. `cap sync` is required after every web build before testing on native.
