---
name: ionic-vue
description: Build a production-ready Ionic Capacitor mobile app with Vue 3 Composition API, composables for cross-cutting concerns, @ionic/vue-router, and vue-i18n. Trigger when the user wants to create an Ionic Vue app, scaffold pages (onboarding, paywall, tabs, settings), or wire RevenueCat / AdMob / push notifications into a Vue Ionic project.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Ionic + Vue App Guidelines

> **This is a SKILL file, NOT a project.** Never run `npm install` here. When creating a new project, ask for the project path or scaffold in a separate directory (e.g. `~/Projects/<app>`).

## When to consult these references

Project setup:

- **Create a new Vue Ionic project**: [new-app.md](references/new-app.md)
- **Project structure**: [project-structure.md](references/project-structure.md)
- **App configuration** (`main.ts`, `App.vue`): [app-config.md](references/app-config.md)
- **Routing** (`@ionic/vue-router` + `beforeEach` guard): [routing.md](references/routing.md)

Pages and navigation:

- **Onboarding guard** (router `beforeEach`): [onboarding-guard.md](references/onboarding-guard.md)
- **Onboarding page** (video background + Swiper): [onboarding-page.md](references/onboarding-page.md)
- **Paywall page** (RevenueCat): [paywall-page.md](references/paywall-page.md)
- **Tabs layout** (with banner ad lifecycle): [tabs-navigation.md](references/tabs-navigation.md)
- **Settings page**: [settings-page.md](references/settings-page.md)

Cross-cutting:

- **Composables** (Theme / Onboarding / Ads / Purchases / Notifications): [composables.md](references/composables.md)
- **i18n with `vue-i18n`**: [i18n-vue-i18n.md](references/i18n-vue-i18n.md)
- **Best practices** (Composition API + `<ion-page>`): [best-practices.md](references/best-practices.md)
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

Use `ion-tabs` + `ion-tab-bar`. Never build a custom tab bar or pull in a third-party tab library.

## Required libraries

```bash
npm install \
  @capacitor/preferences @capacitor/push-notifications \
  @capacitor/splash-screen @capacitor/status-bar \
  @revenuecat/purchases-capacitor @capacitor-community/admob \
  vue-i18n \
  swiper
```

## Hard rules (Vue-specific)

- ❌ Options API for new code — use the **Composition API with `<script setup lang="ts">`**.
- ❌ Direct DOM manipulation — use refs / reactivity.
- ❌ Importing from `@ionic/angular` or `@ionic/react` — only `@ionic/vue`.
- ❌ `any` type.
- ✅ Wrap every page in `<ion-page>` so Ionic's transitions and lifecycle work.

## Before reporting done

```bash
npm install
npm run build
npx cap sync
```

The build must complete without errors. `cap sync` is required after every web build before testing on native.
