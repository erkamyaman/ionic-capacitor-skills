# Create a new Ionic + React app

## Before scaffolding

Confirm the bundle ID with the user (e.g. `com.company.appname`).

## Scaffold

```bash
npm install -g @ionic/cli
ionic start <app-name> blank --type=react --capacitor
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

For React, `webDir` is `dist` (Vite default — verify against your build output).

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
  react-i18next i18next i18next-http-backend \
  swiper
```

## Post-creation cleanup

The `ionic start blank` template includes a default page that conflicts. After scaffolding:

1. Delete the generated default page in `src/pages/` (if any).
2. Replace `App.tsx` with the routing layout in [app-config.md](app-config.md).
3. Build the new directory layout described in [project-structure.md](project-structure.md).

See also: [`../../ionic-shared/references/capacitor-config.md`](../../ionic-shared/references/capacitor-config.md) for cross-framework Capacitor notes.
