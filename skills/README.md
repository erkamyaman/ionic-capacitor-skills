# Ionic Capacitor Skills

Skills for building production-ready Ionic Capacitor mobile apps with **Angular**, **React**, or **Vue**. Each framework skill is self-contained and links into a shared cross-framework skill for native concerns (RevenueCat, AdMob, push notifications, etc.).

## Available Skills

- **`ionic-angular`** — Build an Ionic + Angular app with standalone components, lazy-loaded routes, services, and `@ngx-translate/core` for i18n.
- **`ionic-react`** — Build an Ionic + React app with functional components, hooks, `react-router`, and `react-i18next`.
- **`ionic-vue`** — Build an Ionic + Vue 3 app with Composition API, composables, `@ionic/vue-router`, and `vue-i18n`.
- **`ionic-shared`** — Framework-agnostic Capacitor concerns: AdMob, RevenueCat, push notifications, `@capacitor/preferences` storage, theme, localization content, and store-submission notes. Referenced by all three framework skills.

## Layout

Each skill follows the structure used by [`angular/angular`'s dev-skills](https://github.com/angular/angular/tree/main/skills/dev-skills):

```
<skill-name>/
├── SKILL.md          ← short orientation + links to references/
└── references/
    ├── <topic>.md    ← one focused topic per file
    └── ...
```

`SKILL.md` is a router. The agent loads only the references/ files it needs for the task.

## Using these skills

Activate the relevant skill in your agentic coding tool:

- "Build an Ionic Angular app" → `ionic-angular`
- "Add RevenueCat to my Ionic React app" → `ionic-react` + `ionic-shared/references/revenuecat.md`
- "Set up AdMob in Vue" → `ionic-vue` + `ionic-shared/references/admob.md`
