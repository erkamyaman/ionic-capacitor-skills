---
name: ionic-angular
description: Build a production-ready Ionic Capacitor mobile app with Angular standalone components, lazy-loaded routes, services for cross-cutting concerns, and @ngx-translate/core for i18n. Trigger when the user wants to create an Ionic Angular app, scaffold pages (onboarding, paywall, tabs, settings), or wire RevenueCat / AdMob / push notifications into an Angular Ionic project.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Ionic + Angular App Guidelines

> **This is a SKILL file, NOT a project.** Never run `npm install` here. When creating a new project, ask for the project path or scaffold in a separate directory (e.g. `~/Projects/<app>`).

## When to consult these references

Project setup:

- **Create a new Angular Ionic project**: [new-app.md](references/new-app.md)
- **Project structure** (folder layout): [project-structure.md](references/project-structure.md)
- **App configuration** (`main.ts`, `app.config.ts`): [app-config.md](references/app-config.md)
- **Routing** (lazy `loadComponent`, tabs): [routing.md](references/routing.md)

Pages and navigation:

- **Onboarding guard** (`CanActivateFn`): [onboarding-guard.md](references/onboarding-guard.md)
- **Onboarding page** (video background + Swiper): [onboarding-page.md](references/onboarding-page.md)
- **Paywall page** (RevenueCat): [paywall-page.md](references/paywall-page.md)
- **Tabs navigation**: [tabs-navigation.md](references/tabs-navigation.md)
- **Settings page**: [settings-page.md](references/settings-page.md)

Cross-cutting:

- **Signals** (`signal`, `computed`, `linkedSignal`, `resource`, signal inputs/outputs): [signals.md](references/signals.md)
- **Services** (Theme / Onboarding / Ads / Purchases / Notifications): [services.md](references/services.md)
- **i18n with `@ngx-translate/core`**: [i18n-ngx-translate.md](references/i18n-ngx-translate.md)
- **Best practices** (standalone, separate files, signals-first): [best-practices.md](references/best-practices.md)
- **Commands** (build / sync / open): [commands.md](references/commands.md)

Native plugin topics live in `../ionic-shared/references/` — always link there for AdMob, RevenueCat, push notifications, storage, and theme:

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
  @ngx-translate/core @ngx-translate/http-loader \
  swiper
```

## Hard rules (Angular-specific)

- ❌ NgModules for new pages/components — use **standalone** components.
- ❌ `IonicModule` in standalone components — import the individual components (`IonContent`, `IonButton`, …).
- ❌ Inline `template` or `styles` in `@Component` — always use **separate** `.html`, `.ts`, `.scss` files via `templateUrl` and `styleUrls`.
- ❌ `@angular/http` (deprecated) — use `@angular/common/http`.
- ❌ `any` type — use real TypeScript types.
- ✅ **Prefer Signals** for reactive state (`signal()`, `computed()`, `linkedSignal()`, `resource()`). Services should expose readonly signals where it makes sense — see [signals.md](references/signals.md).
- ✅ Use **signal inputs / outputs** (`input()`, `input.required()`, `output()`) for new components.

## Before reporting done

```bash
npm install
npx ng build
npx cap sync
```

The build must complete without errors. `cap sync` is required after every web build before testing on native.
