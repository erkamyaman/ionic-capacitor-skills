# Create a new Ionic + Vue app

## Before scaffolding

Confirm the bundle ID with the user (e.g. `com.company.appname`).

## Scaffold

```bash
npm install -g @ionic/cli
ionic start <app-name> blank --type=vue --capacitor
cd <app-name>
```

## Capacitor config

Edit `capacitor.config.ts`:

```typescript
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.company.appname',
  appName: 'App Name',
  webDir: 'dist',
  server: { androidScheme: 'https' },
};

export default config;
```

For Vue, `webDir` is `dist` (Vite default — verify against your build output).

## Add native platforms

```bash
npx cap add ios
npx cap add android
```

## Install required dependencies

```bash
npm install \
  @capacitor/preferences @capacitor/push-notifications \
  @capacitor/splash-screen @capacitor/status-bar \
  @revenuecat/purchases-capacitor @capacitor-community/admob \
  vue-i18n \
  swiper
```

## Post-creation cleanup

The `ionic start blank` template includes a default page that conflicts. After scaffolding:

1. Delete the generated default view in `src/views/` (if any).
2. Replace `src/router/index.ts` with the routes in [routing.md](routing.md).
3. Replace `src/main.ts` per [app-config.md](app-config.md).
4. Build the new directory layout described in [project-structure.md](project-structure.md).

See also: [`../../ionic-shared/references/capacitor-config.md`](../../ionic-shared/references/capacitor-config.md) for cross-framework Capacitor notes.
