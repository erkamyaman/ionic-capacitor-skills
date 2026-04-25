# Ionic Capacitor Skills

Skills for building production-ready Ionic Capacitor mobile apps with **Angular**, **React**, or **Vue**.

## Layout

```
<skill-name>/
├── SKILL.md          ← short orientation + links to references/
└── references/
    ├── <topic>.md    ← one focused topic per file
    └── ...
```

`SKILL.md` is a short router with frontmatter. The agent loads only the references/ files it needs for the task.

## Skills by category

### Core

- **`ionic-angular`** — Angular standalone components, lazy routes, Signals, services, `@ngx-translate/core`.
- **`ionic-react`** — React functional components, hooks, `@ionic/react-router`, `react-i18next`.
- **`ionic-vue`** — Vue 3 Composition API, composables, `@ionic/vue-router`, `vue-i18n`.
- **`ionic-shared`** — cross-framework Capacitor concerns: AdMob, RevenueCat, push, storage, theme, localization content, env vars / API keys, store-submission notes.

### Compliance (required for ad/IAP apps)

- **`ionic-cmp-consent`** — Google UMP consent (GDPR / IAB TCF) before AdMob.
- **`ionic-att`** — iOS App Tracking Transparency before personalized ads.

### Release essentials

- **`ionic-app-icon-splash`** — `@capacitor/assets` icon + splash generation.
- **`ionic-deep-links`** — custom URL schemes + Universal Links + App Links.
- **`ionic-apple-sign-in`** — required by Apple alongside other social logins on iOS.

### Native plugins

- **`ionic-native-essentials`** — Camera, Filesystem, Share, Haptics, Network, Keyboard.
- **`ionic-biometric-auth`** — Face ID / Touch ID / fingerprint.
- **`ionic-local-notifications`** — device-scheduled reminders.

### Backend / auth

- **`ionic-firebase`** — Firebase Auth + Firestore.
- **`ionic-supabase`** — Supabase Auth + Postgres + RLS + realtime.

### Operations

- **`ionic-sentry`** — crash + error reporting.
- **`ionic-analytics`** — Firebase Analytics or PostHog.
- **`ionic-in-app-review`** — native rating prompt.

## Using these skills

Activate the relevant skill in your agentic coding tool:

- "Build an Ionic Angular app" → `ionic-angular`
- "Add RevenueCat to my Ionic React app" → `ionic-react` + `ionic-shared/references/revenuecat.md`
- "Add GDPR consent to my AdMob app" → `ionic-cmp-consent`
- "Set up Firebase Auth with Apple sign-in" → `ionic-firebase` + `ionic-apple-sign-in`
