# Sentry Setup

## Sentry dashboard

1. Create a project at <https://sentry.io>. Pick **Capacitor** as the platform.
2. Copy the DSN (`https://<key>@oXXX.ingest.sentry.io/XXX`).
3. Note the org slug and project slug — needed for source map uploads.

## Env vars

DSN is publishable — safe to ship. Per [`../../ionic-shared/references/environments-and-keys.md`](../../ionic-shared/references/environments-and-keys.md):

```env
VITE_SENTRY_DSN=https://abc@o123.ingest.sentry.io/456
```

## Init — Angular

`Sentry.init` must run **before** `bootstrapApplication` so framework integrations (router tracing, error handler) are wired up. The `appConfig` providers below reference `SentryAngular.TraceService`, which `APP_INITIALIZER` pulls into the DI graph after bootstrap.

```typescript
// main.ts
import * as Sentry from '@sentry/capacitor';
import * as SentryAngular from '@sentry/angular';
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

Sentry.init(
  {
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,                    // 'development' | 'production'
    release: 'app@' + (import.meta.env.VITE_APP_VERSION ?? 'dev'),
    tracesSampleRate: 0.1,                                // 10% performance sampling in prod
    replaysSessionSampleRate: 0,
    replaysOnErrorSampleRate: 1.0,
    integrations: [
      SentryAngular.browserTracingIntegration(),
      SentryAngular.replayIntegration(),
    ],
  },
  SentryAngular.init,
);

bootstrapApplication(AppComponent, appConfig);
```

Then provide the error handler in `app.config.ts`:

```typescript
import { ErrorHandler, APP_INITIALIZER, inject } from '@angular/core';
import * as SentryAngular from '@sentry/angular';
import { Router } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    // ... existing providers
    { provide: ErrorHandler, useValue: SentryAngular.createErrorHandler({ showDialog: false }) },
    { provide: SentryAngular.TraceService, deps: [Router] },
    { provide: APP_INITIALIZER, useFactory: () => () => inject(SentryAngular.TraceService), multi: true },
  ],
};
```

## Init — React

```tsx
// main.tsx
import * as Sentry from '@sentry/capacitor';
import * as SentryReact from '@sentry/react';
import { createRoot } from 'react-dom/client';
import App from './App';

Sentry.init(
  {
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    release: 'app@' + (import.meta.env.VITE_APP_VERSION ?? 'dev'),
    tracesSampleRate: 0.1,
    integrations: [SentryReact.browserTracingIntegration()],
  },
  SentryReact.init,
);

const root = createRoot(document.getElementById('root')!);
root.render(
  <SentryReact.ErrorBoundary fallback={<p>Something went wrong</p>}>
    <App />
  </SentryReact.ErrorBoundary>,
);
```

## Init — Vue

```typescript
// main.ts
import { createApp } from 'vue';
import * as Sentry from '@sentry/capacitor';
import * as SentryVue from '@sentry/vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);

Sentry.init(
  {
    app,                                         // pass the Vue app instance
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    release: 'app@' + (import.meta.env.VITE_APP_VERSION ?? 'dev'),
    tracesSampleRate: 0.1,
    integrations: [
      SentryVue.browserTracingIntegration({ router }),
    ],
  },
  SentryVue.init,
);

app.use(router).mount('#app');
```

## Manual error reporting

`@sentry/capacitor` re-exports the framework SDK's runtime API (`captureException`, `captureMessage`, `setUser`, etc.) — always import from `@sentry/capacitor` in app code, never from `@sentry/angular` / `@sentry/react` / `@sentry/vue` directly. This keeps the native iOS/Android crash bridge wired up.

```typescript
import * as Sentry from '@sentry/capacitor';

try {
  await riskyAction();
} catch (err) {
  Sentry.captureException(err, {
    tags: { feature: 'paywall' },
    extra: { selectedPlan },
  });
  throw err;
}

// Custom messages
Sentry.captureMessage('Payment cancelled by user', 'info');

// Add user context (after login)
Sentry.setUser({ id: userId });

// Clear on sign out
Sentry.setUser(null);
```

## Source maps

Without source maps, your JS errors look like:

```
TypeError: Cannot read property 'x' of undefined
  at a.b (https://.../assets/index-abc123.js:1:48372)
```

With source maps:

```
TypeError: Cannot read property 'x' of undefined
  at SettingsPage.handleClick (src/pages/SettingsPage.tsx:42:12)
```

Configure the Sentry CLI to upload maps as part of the build:

```bash
npm install --save-dev @sentry/cli
npx sentry-cli login
```

```json
// package.json
{
  "scripts": {
    "build": "vite build && sentry-cli sourcemaps inject ./dist && sentry-cli sourcemaps upload --release=app@$npm_package_version ./dist"
  }
}
```

For Vite, [`@sentry/vite-plugin`](https://www.npmjs.com/package/@sentry/vite-plugin) automates this (preferred over the CLI for Vite-based React/Vue projects).

For Angular CLI, use [`@sentry/webpack-plugin`](https://www.npmjs.com/package/@sentry/webpack-plugin) via a custom builder, or run the Sentry CLI step in `postbuild`.

## Native crash symbolication

For native iOS / Android crashes to be readable, upload dSYMs (iOS) and ProGuard mappings (Android) during the build:

```bash
sentry-cli debug-files upload --include-sources ios/App/build/...
sentry-cli upload-proguard android/app/build/outputs/mapping/release/mapping.txt
```

Most teams do this as part of the Fastlane lane.

## Performance / replay

`tracesSampleRate: 0.1` samples 10% of transactions for performance. Bump in dev (`1.0`) and lower in prod to manage cost.

`replaysOnErrorSampleRate: 1.0` records the last few seconds of session whenever an error is captured — invaluable for reproducing UI bugs.
