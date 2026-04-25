# Angular Project Structure

```
project-root/
├── src/
│   ├── app/
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   ├── app.config.ts
│   │   ├── app.routes.ts
│   │   ├── tabs/
│   │   │   ├── tabs.page.ts
│   │   │   ├── tabs.page.html
│   │   │   ├── tabs.page.scss
│   │   │   └── tabs.routes.ts
│   │   ├── home/
│   │   │   ├── home.page.ts
│   │   │   ├── home.page.html
│   │   │   └── home.page.scss
│   │   ├── explore/
│   │   ├── settings/
│   │   ├── paywall/
│   │   ├── onboarding/
│   │   ├── services/
│   │   │   ├── theme.service.ts
│   │   │   ├── onboarding.service.ts
│   │   │   ├── ads.service.ts
│   │   │   ├── purchases.service.ts
│   │   │   └── notifications.service.ts
│   │   ├── guards/
│   │   │   └── onboarding.guard.ts
│   │   └── utils/
│   │       ├── admob.ts
│   │       ├── purchases.ts
│   │       ├── onboarding.ts
│   │       ├── theme.ts
│   │       └── notifications.ts
│   ├── assets/
│   │   └── i18n/
│   │       ├── en.json
│   │       └── tr.json
│   ├── theme/
│   │   └── variables.scss
│   ├── global.scss
│   ├── index.html
│   └── main.ts
├── ios/
├── android/
├── capacitor.config.ts
├── angular.json
├── package.json
└── tsconfig.json
```

## Key conventions

- **One folder per page**, each with `.page.ts`, `.page.html`, `.page.scss`.
- **`utils/`** holds framework-agnostic logic (just Capacitor calls, no Angular). The cross-framework `ionic-shared` skill defines these utilities — they're identical across Angular / React / Vue.
- **`services/`** holds the Angular-flavored wrappers around `utils/` — see [services.md](services.md).
- **`guards/`** for functional route guards (`CanMatchFn` for the onboarding gate) — see [onboarding-guard.md](onboarding-guard.md).
- **`assets/i18n/`** for translation JSON loaded by `@ngx-translate/http-loader`.
