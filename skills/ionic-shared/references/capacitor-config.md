# Capacitor Config & Native Platforms

## `capacitor.config.ts`

```typescript
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.company.appname',
  appName: 'App Name',
  webDir: 'www',
  server: {
    androidScheme: 'https',
  },
};

export default config;
```

`webDir` defaults:

| Framework | `webDir` |
|-----------|----------|
| Angular   | `www`    |
| React     | `dist`   |
| Vue       | `dist`   |

Always confirm the build output directory in the project's build config and align `webDir` with it before the first `cap sync`.

## Adding native platforms

```bash
npx cap add ios
npx cap add android
```

## Sync after every web build

```bash
npx cap sync
```

`cap sync` copies the built web assets into the native iOS/Android projects and updates plugin bindings. It must run after every build before testing on device.

## Opening native projects

```bash
npx cap open ios       # opens Xcode
npx cap open android   # opens Android Studio
```

## Bundle ID

Always confirm the bundle ID with the user before scaffolding (e.g., `com.company.appname`). The bundle ID is used as the `appId` in `capacitor.config.ts` and must match the App Store Connect / Play Console listing.
