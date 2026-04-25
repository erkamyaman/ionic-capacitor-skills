# Vue Project Structure

```
project-root/
├── src/
│   ├── App.vue
│   ├── main.ts
│   ├── router/
│   │   └── index.ts
│   ├── views/
│   │   ├── OnboardingPage.vue
│   │   ├── PaywallPage.vue
│   │   ├── SettingsPage.vue
│   │   ├── HomePage.vue
│   │   ├── ExplorePage.vue
│   │   └── TabsLayout.vue
│   ├── composables/
│   │   ├── useTheme.ts
│   │   ├── useOnboarding.ts
│   │   ├── useAds.ts
│   │   ├── usePurchases.ts
│   │   └── useNotifications.ts
│   ├── utils/
│   │   ├── admob.ts
│   │   ├── purchases.ts
│   │   ├── onboarding.ts
│   │   ├── theme.ts
│   │   └── notifications.ts
│   ├── theme/
│   │   └── variables.css
│   └── assets/
│       └── i18n/
│           ├── en.json
│           └── tr.json
├── ios/
├── android/
├── capacitor.config.ts
├── package.json
└── tsconfig.json
```

## Key conventions

- **`views/`** for route-level components (Vue convention), each wrapped in `<ion-page>`.
- **`composables/`** wraps the framework-agnostic `utils/` functions in Vue composables — see [composables.md](composables.md).
- **`utils/`** holds the same Capacitor-only logic shared with Angular and React. See [`../../ionic-shared/`](../../ionic-shared/).
- **`router/index.ts`** holds routes and the onboarding `beforeEach` guard.
- **`assets/i18n/`** holds the `vue-i18n` JSON loaded directly via static imports.
