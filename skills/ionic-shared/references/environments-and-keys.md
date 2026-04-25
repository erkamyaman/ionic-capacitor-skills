# Environments & API Keys

> **A mobile app bundle is not a secret.** Anything compiled into the JavaScript that ships in the IPA / APK can be extracted by anyone with the file. Treat the mobile app as a public client — only **publishable** keys go inside; **secret** keys live behind a backend.

## What's safe to ship in the bundle

| Key type | Examples | Safe in mobile bundle? |
|----------|----------|------------------------|
| RevenueCat **public** SDK key (`appl_…` / `goog_…`) | iOS / Android API keys | ✅ Yes — designed for client use |
| AdMob **ad unit ID** (`ca-app-pub-…/…`) | Banner / interstitial unit IDs | ✅ Yes |
| Firebase web config (apiKey, projectId, …) | Firebase initialization | ✅ Yes — Security Rules do the gating |
| `google-services.json` / `GoogleService-Info.plist` | Native Firebase config files | ✅ Yes — committed alongside source |
| Supabase anon key | Client init | ✅ Yes — RLS policies do the gating |
| Sentry DSN | Crash reporting | ✅ Yes |
| PostHog **project API key** | Analytics ingest | ✅ Yes |
| Mixpanel **project token** | Analytics ingest | ✅ Yes — write-only by design |
| OneSignal **app ID** | Push registration | ✅ Yes (the **REST API key** is server-only) |
| Algolia **search-only** API key | Search-from-client | ✅ Yes (admin key is server-only; consider Secured API Keys for per-user scoping) |
| Mapbox **public access token** (`pk.…`) | Map tiles | ✅ Yes — **but URL-restrict it** in the Mapbox dashboard |
| Google Maps API key | Map tiles / geocoding | ✅ Yes — **but restrict by bundle ID + SHA-1** in the GCP console |
| Stripe **publishable** key (`pk_…`) | Client tokenization | ✅ Yes |
| Stripe **secret** key (`sk_…`) | Server-side only | ❌ NEVER |
| RevenueCat REST **secret** key | Server-to-server calls | ❌ NEVER — proxy through your backend |
| OpenAI / Anthropic API keys | LLM calls | ❌ NEVER — proxy through your backend |
| OneSignal REST API key | Sending notifications | ❌ NEVER |
| Algolia **admin** API key | Index writes / config | ❌ NEVER |
| Firebase **service account** / Admin SDK | Server-side admin | ❌ NEVER |
| Database credentials | Direct DB access | ❌ NEVER |

The rule of thumb: if the key gives an attacker **billing impact** or **read/write access to other users' data**, it doesn't belong in the app.

> **Even publishable keys should be rotated if leaked.** A public GitHub commit gets scraped by bots within minutes. If a publishable key ends up there — even one that "doesn't matter" — rotate it. Especially Mapbox / Google Maps tokens (which can be abused for free tile loads on someone else's bill if URL restrictions aren't set).

## Why env vars still help

Even for "publishable" keys, you don't want to hardcode them in source — different keys for dev / staging / production, key rotation, and not leaking them via screen-shares or repo searches all benefit from environment variables.

## Vite (React / Vue)

Vite reads `.env*` files at build time. Only variables prefixed with `VITE_` are exposed to client code.

| File | Loaded by | Precedence |
|------|-----------|------------|
| `.env.[mode].local` (e.g. `.env.development.local`) | mode-specific local override — gitignored | **highest** |
| `.env.[mode]`       | mode-specific (e.g. `.env.development` for `vite` / `vite build --mode development`; `.env.production` for default `vite build`) | high |
| `.env.local`        | always — gitignored | medium |
| `.env`              | always | **lowest** |

A variable defined in a higher-precedence file overrides the same name in lower files.

```env
# .env.development
VITE_REVENUECAT_KEY_IOS=appl_YOUR_DEV_IOS_KEY
VITE_REVENUECAT_KEY_ANDROID=goog_YOUR_DEV_ANDROID_KEY
VITE_ADMOB_BANNER_ID=ca-app-pub-3940256099942544/6300978111
```

Read at runtime:

```typescript
const apiKey = Capacitor.getPlatform() === 'ios'
  ? import.meta.env.VITE_REVENUECAT_KEY_IOS
  : import.meta.env.VITE_REVENUECAT_KEY_ANDROID;
```

`.env.local` MUST be gitignored. The other `.env*` files are typically committed (so CI can build), but **only when they contain publishable keys**.

### `.env.example` (commit this)

```env
# .env.example — copy to .env.local and fill in
VITE_REVENUECAT_KEY_IOS=appl_xxxxxxxxxxxxxxxxxxxx
VITE_REVENUECAT_KEY_ANDROID=goog_xxxxxxxxxxxxxxxxxxxx
VITE_ADMOB_BANNER_ID=ca-app-pub-XXXXXXXXXXXXXXXX/YYYYYYYYYY
VITE_SENTRY_DSN=https://xxx@oxxx.ingest.sentry.io/xxx
VITE_APP_VERSION=0.1.0          # used by Sentry releases — match package.json
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_PROJECT_ID=
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_POSTHOG_KEY=
```

### Vite-built-in `import.meta.env` values

Vite injects two boolean/string flags into every build:

| Variable | Type | Value |
|----------|------|-------|
| `import.meta.env.MODE` | `string` | `'development'` in `vite` dev / `vite build --mode development`, `'production'` in default `vite build`. |
| `import.meta.env.DEV`  | `boolean` | `true` in development mode, `false` in production. |
| `import.meta.env.PROD` | `boolean` | Inverse of `DEV`. |
| `import.meta.env.SSR`  | `boolean` | `true` in SSR builds (n/a for Capacitor). |

Use `import.meta.env.DEV` for boolean toggles (typo-proof) and `import.meta.env.MODE` when you need the literal string (e.g., Sentry's `environment: import.meta.env.MODE`). Both are populated by Vite — no `.env` entry required.

### `VITE_APP_VERSION` from `package.json`

For `VITE_APP_VERSION`, the cleanest pattern is to inject it at build time from `package.json` so it's always in sync. Vite supports this natively:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import pkg from './package.json';   // requires "resolveJsonModule": true in tsconfig.json

export default defineConfig({
  define: {
    'import.meta.env.VITE_APP_VERSION': JSON.stringify(pkg.version),
  },
});
```

If `tsc` errors on the `package.json` import, set `"resolveJsonModule": true` in `tsconfig.json` (it's already enabled in default Ionic + Vite scaffolds, but worth checking).

That removes the need to keep `VITE_APP_VERSION` in env files at all — it always matches `package.json`.

## Angular CLI

Two options — pick one and be consistent.

### Option A: `src/environments/` + file replacements (recommended)

The Angular CLI's `ng generate environments` scaffolds `environment.ts` and `environment.prod.ts` (note: `.prod.ts` is the default convention, not `.production.ts` — though the actual filename is arbitrary as long as `angular.json` `fileReplacements` matches).

```typescript
// src/environments/environment.ts (development)
export const environment = {
  production: false,
  revenueCat: {
    iosKey: 'appl_YOUR_DEV_IOS_KEY',
    androidKey: 'goog_YOUR_DEV_ANDROID_KEY',
  },
  admob: {
    bannerId: 'ca-app-pub-3940256099942544/6300978111',
    isTesting: true,
  },
};
```

```typescript
// src/environments/environment.prod.ts
export const environment = {
  production: true,
  revenueCat: {
    iosKey: 'appl_YOUR_PROD_IOS_KEY',
    androidKey: 'goog_YOUR_PROD_ANDROID_KEY',
  },
  admob: {
    bannerId: 'ca-app-pub-XXXXXXXXXXXXXXXX/YYYYYYYYYY',
    isTesting: false,
  },
};
```

`angular.json` `fileReplacements` swaps `environment.ts` → `environment.prod.ts` on `--configuration production` builds. Both files **must not contain real secrets** — they're committed to source control.

For per-developer overrides without source-control noise, add `src/environments/environment.local.ts` and gitignore it.

### Option B: `.env` parity via `@ngx-env/builder`

If your team prefers `.env`-style configuration to match Vite's pattern (e.g., a monorepo with mixed Angular + React projects), [`@ngx-env/builder`](https://www.npmjs.com/package/@ngx-env/builder) is the well-trodden path as of writing — it plugs into the Angular CLI build and exposes `process.env.NG_APP_*` at build time. Verify it's still maintained against your Angular version before adopting; the Angular CLI itself has been adding more native `.env` support over time, so this third-party builder may become unnecessary.

Don't roll your own dotenv setup with custom builders unless you have a specific reason.

## .gitignore

Make sure these are ignored:

```
.env
.env.local
.env.*.local
src/environments/environment.local.ts
```

`.env.development` / `.env.production` may or may not be committed depending on whether they hold real keys vs placeholders. When in doubt, gitignore them and commit a `.env.example` documenting the expected variables.

## Test vs production toggles

The AdMob and RevenueCat utilities have `isTesting: true` / `initializeForTesting: true` flags. These MUST be flipped to `false` in production builds. Tie them to the build mode, not a hand-set string env var:

```typescript
// utils/admob.ts (sketch) — Vite (React / Vue)
import { Capacitor } from '@capacitor/core';

// import.meta.env.DEV is `true` in `vite` dev / development mode, `false` in production builds.
// Boolean — typo-proof, always set, requires no env file.
const ADMOB_TESTING = import.meta.env.DEV;

await AdMob.initialize({ initializeForTesting: ADMOB_TESTING });
```

```typescript
// utils/admob.ts — Angular
import { environment } from '../environments/environment';

const ADMOB_TESTING = !environment.production;

await AdMob.initialize({ initializeForTesting: ADMOB_TESTING });
```

Why not a string env var like `VITE_ADMOB_TESTING=true`? Because typos silently fail (`'True'` !== `'true'`) — and the failure direction matters: if a typo means the flag stays `false`, you ship test ads to production and the AdMob account gets flagged. `import.meta.env.DEV` and `environment.production` are booleans you can't mistype.

## Server-side secrets — what to do instead

If the app needs to call OpenAI / Anthropic / a payment processor / a database directly, **don't**. Set up a backend (Cloud Function, Worker, Express, etc.) that:

1. Accepts a request from the authenticated app.
2. Holds the secret server-side.
3. Calls the upstream API on the user's behalf.
4. Returns the result.

This adds ~50 lines of code and is the only way to keep a real secret.

## Summary checklist

- [ ] Publishable keys live in env files, not source.
- [ ] `.env.local` and any file with real prod keys is gitignored.
- [ ] `.env.example` is committed to document the expected variables.
- [ ] Test ad IDs in dev builds; production ad IDs in release builds.
- [ ] `initializeForTesting` / `isTesting` flags are tied to `import.meta.env.DEV` (Vite) or `!environment.production` (Angular) — never a hand-set string.
- [ ] Mapbox / Google Maps tokens are URL- or bundle-restricted in their respective dashboards.
- [ ] No secret keys (Stripe `sk_`, OpenAI / Anthropic, RevenueCat REST secret, OneSignal REST key, Algolia admin key) anywhere in the bundle.
- [ ] A server-side proxy exists for any call that requires a secret.
- [ ] A leaked publishable key — even one that "shouldn't matter" — gets rotated.
