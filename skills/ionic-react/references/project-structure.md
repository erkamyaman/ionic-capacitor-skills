# React Project Structure

```
project-root/
├── src/
│   ├── App.tsx
│   ├── main.tsx
│   ├── pages/
│   │   ├── OnboardingPage.tsx
│   │   ├── OnboardingPage.css       # background-video + gradient-overlay styles
│   │   ├── PaywallPage.tsx
│   │   ├── SettingsPage.tsx
│   │   ├── HomePage.tsx
│   │   └── ExplorePage.tsx
│   ├── components/
│   │   ├── TabsLayout.tsx
│   │   └── OnboardingGuard.tsx
│   ├── hooks/
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
│   └── i18n/
│       ├── index.ts
│       ├── en.json
│       └── tr.json
├── public/
├── ios/
├── android/
├── capacitor.config.ts
├── package.json
└── tsconfig.json
```

## Key conventions

- **`pages/`** for route-level components, each wrapped in `<IonPage>`.
- **`components/`** for shared UI (`TabsLayout`, `OnboardingGuard`).
- **`hooks/`** wraps the framework-agnostic `utils/` functions in React hooks — see [hooks.md](hooks.md).
- **`utils/`** holds the same Capacitor-only logic shared with Angular and Vue. See [`../../ionic-shared/`](../../ionic-shared/).
- **`i18n/`** holds the `react-i18next` setup plus `en.json` / `tr.json`.
