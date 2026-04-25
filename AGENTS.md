# AGENTS.md

Guidance for AI agents and contributors working on this skills repository.

## Quick start

```bash
# This is a skills repository — it contains SKILL.md files, NOT runnable code.
# NEVER run npm install, ionic serve, or ng build here.
# Skills are installed into target projects via:
npx skills add erkamyaman/ionic-skills
```

## Repository structure

```
ionic-skills/
├── .gitignore
├── AGENTS.md                 # ← you are here
├── README.md                 # public docs
├── package.json              # skills registry (skills array)
└── skills/
    ├── README.md             # skills index
    │
    ├── ionic-shared/         # framework-agnostic Capacitor concerns (referenced by all 3 frameworks)
    │   ├── SKILL.md
    │   └── references/       # capacitor-config, environments-and-keys, storage, theming,
    │                         # admob, revenuecat, push-notifications, localization-content,
    │                         # app-store-notes, testing-checklist
    │
    ├── ionic-angular/        # Ionic + Angular: standalone components, Signals, services
    │   ├── SKILL.md
    │   └── references/       # new-app, project-structure, app-config, routing, services,
    │                         # signals, control-flow, accessibility, defer-loading,
    │                         # onboarding-guard, onboarding-page, paywall-page,
    │                         # tabs-navigation, settings-page, i18n-ngx-translate,
    │                         # best-practices, commands
    │
    ├── ionic-react/          # Ionic + React: functional components, hooks, react-i18next
    │   ├── SKILL.md
    │   └── references/       # new-app, project-structure, app-config, routing, hooks,
    │                         # onboarding-guard, onboarding-page, paywall-page,
    │                         # tabs-navigation, settings-page, i18n-react-i18next,
    │                         # best-practices, commands
    │
    ├── ionic-vue/            # Ionic + Vue 3 Composition API, composables, vue-i18n
    │   ├── SKILL.md
    │   └── references/       # new-app, project-structure, app-config, routing, composables,
    │                         # onboarding-guard, onboarding-page, paywall-page,
    │                         # tabs-navigation, settings-page, i18n-vue-i18n,
    │                         # best-practices, commands
    │
    │   # Compliance (required for ad/IAP apps)
    ├── ionic-cmp-consent/    # Google UMP — GDPR consent before AdMob
    ├── ionic-att/            # iOS App Tracking Transparency
    │
    │   # Release essentials
    ├── ionic-app-icon-splash/   # @capacitor/assets — generate icons + splash
    ├── ionic-deep-links/        # custom schemes + Universal Links + App Links
    ├── ionic-apple-sign-in/     # Sign in with Apple (use only outside Firebase)
    │
    │   # Native plugins
    ├── ionic-native-essentials/ # Camera/Filesystem/Share/Haptics/Network/Keyboard
    ├── ionic-biometric-auth/    # Face ID / Touch ID / fingerprint
    ├── ionic-local-notifications/  # device-scheduled reminders
    │
    │   # Backend / auth
    ├── ionic-firebase/       # Firebase Auth + Firestore + native social sign-in
    ├── ionic-supabase/       # Supabase Auth + Postgres + RLS + realtime
    │
    │   # Operations
    ├── ionic-sentry/         # crash + error reporting
    ├── ionic-analytics/      # Firebase Analytics or PostHog
    └── ionic-in-app-review/  # native rating prompt
```

Every skill follows the same layout: `<skill>/SKILL.md` (router + frontmatter) plus `<skill>/references/*.md` (one focused topic per file). The single-file skills (`ionic-in-app-review`, parts of `ionic-biometric-auth`) collapse references into the SKILL.md when there's only one topic.

## How skills work

- Each skill lives in `skills/<skill-name>/`.
- `SKILL.md` carries YAML frontmatter (`name`, `description`, `license`, `metadata.author`, `metadata.version`) and acts as a **router** — short orientation plus links into `references/`.
- `references/*.md` files cover one focused topic each so the agent loads only what's relevant.
- Cross-framework topics live in `ionic-shared/references/` and are linked from each framework skill's `SKILL.md`.
- `package.json`'s `skills` array enumerates the available skills.

## Adding a new skill

1. Create the folder:

   ```bash
   mkdir -p skills/my-new-skill/references
   ```

2. Create `skills/my-new-skill/SKILL.md` with frontmatter:

   ```markdown
   ---
   name: my-new-skill
   description: One-sentence trigger description so agents know when to load this.
   license: MIT
   metadata:
     author: erkamyaman
     version: '1.0'
   ---

   # My New Skill

   Short orientation. Then links into `references/`.
   ```

3. Add focused topic files in `references/<topic>.md`.
4. Add the skill name to the `skills` array in `package.json`.
5. Add a row to the table in `README.md` and to `skills/README.md`.

## Editing existing skills

- Update the relevant `references/<topic>.md` — that's where the meat lives.
- Update `SKILL.md` only if the orientation, link list, or hard rules change.
- Bump `metadata.version` in the frontmatter when a change is significant.
- Keep code examples accurate — agents copy them directly into projects.
- Test code examples in a real Ionic Capacitor project for the affected framework before committing.

## Skill writing guidelines

### Do

- Write clear imperative instructions ("ALWAYS", "NEVER", "MUST").
- Keep each `references/<topic>.md` focused on one topic, with full copy-pasteable code (including imports).
- Specify exact package names.
- List forbidden patterns alongside the correct alternative.
- For each framework's `references/`, link to `../../ionic-shared/references/<topic>.md` rather than duplicate cross-framework content.

### Don't

- Include runnable code in this repository.
- Reference files that exist only inside generated projects.
- Use vague language ("consider using", "you might want to").
- Skip imports in code examples.
- Duplicate the same RevenueCat / AdMob / push code in three skills — link to `ionic-shared` instead.

## Pull request guidelines

Every PR must include:

1. **What** — what does this PR change?
2. **Why** — why is it needed?
3. **How** — implementation approach.
4. **Testing** — how the skill was verified (which framework, what was scaffolded, what was built).

### Rules

- All code examples must work in a real project for the framework they target.
- Keep skills focused — one skill per topic area; one topic per `references/` file.
- AI-authored PRs are welcome — be transparent about it.
- Use correct Turkish characters in any TR localization examples (`ı İ ü ö ç ş ğ`).

## Common pitfalls

- Do NOT run `npm install` or build commands in this repo — it's not a project.
- Use `@capacitor/preferences` — never `localStorage` or `@ionic/storage`.
- Use `@capacitor-community/admob` — never deprecated ad libraries.
- Always show `async/await` for Capacitor plugin calls — they are never synchronous.

### Angular-specific

- Standalone components only — no NgModules for new pages.
- Separate `.html` / `.ts` / `.scss` files via `templateUrl` + `styleUrls` — never inline `template`/`styles`.
- Import individual Ionic components (`IonButton`, `IonContent`) — never `IonicModule`.
- **Prefer Signals** for reactive state — `signal()`, `computed()`, `effect()`. Services should expose state as readonly signals. Stick to **stable** signal APIs; avoid Angular features marked "developer preview" / "experimental" (e.g. `linkedSignal`, `resource`, Signal Forms) until they graduate.

### React-specific

- Functional components with hooks only — no class components.
- Wrap every page in `<IonPage>`.
- Use `@ionic/react` only — never `@ionic/angular` or `@ionic/vue`.

### Vue-specific

- Composition API with `<script setup lang="ts">` only — no Options API.
- Wrap every page in `<ion-page>`.
- `vue-i18n` with `legacy: false`.
- Use `@ionic/vue` only — never `@ionic/angular` or `@ionic/react`.
