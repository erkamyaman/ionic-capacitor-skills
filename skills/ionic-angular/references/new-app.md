# Create a new Ionic + Angular app

## Before scaffolding

Confirm the bundle ID with the user (e.g. `com.company.appname`). It must match what will be registered in App Store Connect / Play Console.

## Scaffold

```bash
npm install -g @ionic/cli
ionic start <app-name> blank --type=angular --capacitor
cd <app-name>
```

## Capacitor config

Edit `capacitor.config.ts`:

```typescript
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.company.appname',
  appName: 'App Name',
  webDir: 'www',
  server: { androidScheme: 'https' },
};

export default config;
```

For Angular, `webDir` is `www` (verify against `angular.json`'s build output).

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
  @ngx-translate/core @ngx-translate/http-loader \
  swiper
```

## Post-creation cleanup

The `ionic start blank` template generates a default `home` page that conflicts with the structure used here. After scaffolding:

1. Delete the generated `src/app/home/` directory (it'll be replaced by the new `home` tab).
2. Replace `app.routes.ts` per [routing.md](routing.md).
3. Update `app.config.ts` per [app-config.md](app-config.md).
4. Build the new directory layout described in [project-structure.md](project-structure.md).

See also: [`../../ionic-shared/references/capacitor-config.md`](../../ionic-shared/references/capacitor-config.md) for cross-framework Capacitor notes.
