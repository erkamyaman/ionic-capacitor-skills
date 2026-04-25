# Ionic Capacitor Skills

A collection of agent skills for building production-ready Ionic Capacitor mobile applications with **Angular**, **React**, or **Vue**.

## Installation

```bash
npx skills add erkamyaman/ionic-skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [ionic-angular](./skills/ionic-angular) | Ionic + Angular standalone components, lazy routes, Signals, services, `@ngx-translate/core` |
| [ionic-react](./skills/ionic-react)     | Ionic + React functional components, hooks, `@ionic/react-router`, `react-i18next` |
| [ionic-vue](./skills/ionic-vue)         | Ionic + Vue 3 Composition API, composables, `@ionic/vue-router`, `vue-i18n` |
| [ionic-shared](./skills/ionic-shared)   | Framework-agnostic Capacitor concerns — AdMob, RevenueCat, push, storage, theme, localization, store-submission notes. Referenced by all three framework skills |

## Layout

Each skill follows the [`angular/angular` dev-skills](https://github.com/angular/angular/tree/main/skills/dev-skills) pattern:

```
skills/<skill-name>/
├── SKILL.md          ← short orientation + links to references/
└── references/
    ├── <topic>.md    ← one focused topic per file
    └── ...
```

`SKILL.md` is a router. The agent loads only the references/ files it needs for the task at hand.

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
| **Onboarding guard**  | `CanActivateFn` (Angular), wrapper component (React), `beforeEach` (Vue) |
| **State**             | Angular services + Signals / React hooks / Vue composables |

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
- **Angular** — `services/` wrap the utilities with `@Injectable({ providedIn: 'root' })`. Premium status, theme, language exposed as **Signals** so templates re-render reactively.
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
