# Ionic Capacitor Skills

A collection of agent skills for building production-ready Ionic Capacitor mobile applications with **Angular**, **React**, or **Vue**.

## Installation

```bash
npx skills add erkamyaman/ionic-skills
```

## Available Skills

### Core (framework scaffolds + shared concerns)

| Skill | Description |
|-------|-------------|
| [ionic-angular](./skills/ionic-angular) | Ionic + Angular standalone components, lazy routes, Signals, services, `@ngx-translate/core` |
| [ionic-react](./skills/ionic-react)     | Ionic + React functional components, hooks, `@ionic/react-router`, `react-i18next` |
| [ionic-vue](./skills/ionic-vue)         | Ionic + Vue 3 Composition API, composables, `@ionic/vue-router`, `vue-i18n` |
| [ionic-shared](./skills/ionic-shared)   | Framework-agnostic Capacitor concerns — AdMob, RevenueCat, push, storage, theme, localization, env vars, store-submission notes. Referenced by all three framework skills |

### Compliance (required for ad/IAP apps)

| Skill | Description |
|-------|-------------|
| [ionic-cmp-consent](./skills/ionic-cmp-consent) | Google UMP consent dialog — required in EU/UK before AdMob serves personalized ads |
| [ionic-att](./skills/ionic-att)                 | iOS App Tracking Transparency — required for personalized ads on iOS 14+; App Review rejects without it |

### Release essentials

| Skill | Description |
|-------|-------------|
| [ionic-app-icon-splash](./skills/ionic-app-icon-splash) | `@capacitor/assets` — generate every iOS/Android icon + splash size from one source PNG |
| [ionic-deep-links](./skills/ionic-deep-links)           | Custom URL schemes + iOS Universal Links + Android App Links |
| [ionic-apple-sign-in](./skills/ionic-apple-sign-in)     | Sign in with Apple — required by Apple if you offer any other social login on iOS |

### Native plugins

| Skill | Description |
|-------|-------------|
| [ionic-native-essentials](./skills/ionic-native-essentials) | Camera, Filesystem, Share, Haptics, Network, Keyboard — six small Capacitor plugins bundled |
| [ionic-biometric-auth](./skills/ionic-biometric-auth)       | Face ID / Touch ID / fingerprint via `@aparajita/capacitor-biometric-auth` |
| [ionic-local-notifications](./skills/ionic-local-notifications) | Device-scheduled reminders via `@capacitor/local-notifications` (distinct from push) |

### Backend / auth

| Skill | Description |
|-------|-------------|
| [ionic-firebase](./skills/ionic-firebase)   | Firebase Auth + Firestore + native social sign-in |
| [ionic-supabase](./skills/ionic-supabase)   | Supabase Auth + Postgres + RLS + realtime |

### Operations

| Skill | Description |
|-------|-------------|
| [ionic-sentry](./skills/ionic-sentry)             | Crash + error reporting via `@sentry/capacitor` + framework SDK |
| [ionic-analytics](./skills/ionic-analytics)       | Product analytics — Firebase Analytics or PostHog, with event taxonomy |
| [ionic-in-app-review](./skills/ionic-in-app-review) | Native App Store / Play Store rating prompt |

## Layout

```
skills/<skill-name>/
├── SKILL.md          ← short orientation + links to references/
└── references/
    ├── <topic>.md    ← one focused topic per file
    └── ...
```

`SKILL.md` is a short router with frontmatter. The agent loads only the references/ files it needs for the task at hand.

## What's Included

| Feature | Implementation |
|---------|----------------|
| **Framework**         | Ionic Framework + Angular / React / Vue (latest stable) |
| **Native Runtime**    | Capacitor (latest stable) |
| **Onboarding**        | Swiper slides over a fullscreen HTML5 video + gradient overlay |
| **Paywall**           | RevenueCat (weekly / yearly with 50% OFF badge) + Restore Purchases |
| **Ads**               | AdMob banner via `@capacitor-community/admob`, hidden for premium users |
| **i18n**              | Turkish / English — `@ngx-translate/core` (Angular), `react-i18next` (React), `vue-i18n` (Vue) |
| **Theming**           | Light / Dark / System via the `ion-palette-dark` class |
| **Navigation**        | `ion-tabs` + framework-specific router |
| **Storage**           | `@capacitor/preferences` |
| **Notifications**     | `@capacitor/push-notifications` |
| **Onboarding guard**  | `CanMatchFn` (Angular), wrapper component (React), `beforeEach` (Vue) |
| **State**             | Angular services exposing readonly Signals (`OnPush`-friendly) / React hooks / Vue composables |

### App flow

```
Onboarding (video bg) → Paywall (subscriptions) → Main App (tabs)
```

### Required pages

- **Onboarding** — fullscreen video background, swipe-based slides
- **Paywall** — weekly/yearly subscription options, restore purchases
- **Settings** — language, theme, notifications, remove ads, reset onboarding
- **Home / Explore / Settings** — three-tab default layout

## Architecture

A **shared utilities + framework wrappers** pattern keeps Capacitor logic in one place:

- **`utils/`** — framework-agnostic TypeScript functions calling Capacitor plugins (AdMob, RevenueCat, Preferences, etc.). Defined once in `ionic-shared`, copied identically into each framework's project.
- **Angular** — `services/` wrap the utilities with `@Injectable({ providedIn: 'root' })` and expose state as readonly **Signals** (signal-store pattern). Pages use `ChangeDetectionStrategy.OnPush`, modern built-in control flow (`@if` / `@for`), and `inject()` for DI. Skill avoids any APIs marked "developer preview" / "experimental" (no `linkedSignal`, `resource`, or Signal Forms until they graduate).
- **React** — `hooks/` wrap the utilities with `useCallback`.
- **Vue** — `composables/` wrap the utilities as functions returning state + actions.

## Tech stack

- [Ionic Framework](https://ionicframework.com)
- [Angular](https://angular.dev) / [React](https://react.dev) / [Vue](https://vuejs.org)
- [Capacitor](https://capacitorjs.com)
- [RevenueCat](https://www.revenuecat.com)
- [@capacitor-community/admob](https://github.com/capacitor-community/admob)

## Contributing

Add a new skill by creating a folder in `skills/<skill-name>/` containing:

- `SKILL.md` — short orientation with frontmatter and a list of references.
- `references/*.md` — focused topic files referenced from `SKILL.md`.

Then add the skill name to the `skills` array in `package.json` and a row in this README's table.

See [AGENTS.md](./AGENTS.md) for the full contributor guide.

## License

MIT
