# Environments & API Keys

> **A mobile app bundle is not a secret.** Anything compiled into the JavaScript that ships in the IPA / APK can be extracted by anyone with the file. Treat the mobile app as a public client — only **publishable** keys go inside; **secret** keys live behind a backend.

## What's safe to ship in the bundle

| Key type | Examples | Safe in mobile bundle? |
|----------|----------|------------------------|
| RevenueCat **public** SDK key (`appl_…` / `goog_…`) | iOS / Android API keys | ✅ Yes — designed for client use |
| AdMob **ad unit ID** (`ca-app-pub-…/…`) | Banner / interstitial unit IDs | ✅ Yes |
| Firebase web config (apiKey, projectId, …) | Firebase initialization | ✅ Yes — Firebase enforces auth via Security Rules |
| Supabase anon key | Client init | ✅ Yes — RLS policies do the gating |
| Sentry DSN | Crash reporting | ✅ Yes |
| Stripe **publishable** key (`pk_…`) | Client tokenization | ✅ Yes |
| Stripe **secret** key (`sk_…`) | Server-side only | ❌ NEVER |
| RevenueCat REST **secret** key | Server-to-server calls | ❌ NEVER — proxy through your backend |
| OpenAI / Anthropic API keys | LLM calls | ❌ NEVER — proxy through your backend |
| Database credentials | Direct DB access | ❌ NEVER |

The rule of thumb: if the key gives an attacker **billing impact** or **read/write access to other users' data**, it doesn't belong in the app.

## Why env vars still help

Even for "publishable" keys, you don't want to hardcode them in source — different keys for dev / staging / production, key rotation, and not leaking them via screen-shares or repo searches all benefit from environment variables.

## Vite (React / Vue)

Vite reads `.env*` files at build time. Only variables prefixed with `VITE_` are exposed to client code.

```
.env.development     # vite dev / vite build --mode development
.env.production      # vite build (production default)
.env.local           # local overrides — gitignored
```

```env
# .env.development
VITE_REVENUECAT_KEY_IOS=appl_YOUR_DEV_IOS_KEY
VITE_REVENUECAT_KEY_ANDROID=goog_YOUR_DEV_ANDROID_KEY
VITE_ADMOB_BANNER_ID=ca-app-pub-3940256099942544/6300978111
VITE_ADMOB_TESTING=true
```

Read at runtime:

```typescript
const apiKey = Capacitor.getPlatform() === 'ios'
  ? import.meta.env.VITE_REVENUECAT_KEY_IOS
  : import.meta.env.VITE_REVENUECAT_KEY_ANDROID;
```

`.env.local` MUST be gitignored. The other `.env*` files are typically committed (so CI can build), but **only when they contain publishable keys**.

## Angular CLI

Two options — pick one and be consistent.

### Option A: `src/environments/` + file replacements (classic Angular)

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
// src/environments/environment.production.ts
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

`angular.json` `fileReplacements` swaps `environment.ts` → `environment.production.ts` on `--configuration production` builds. Both files **must not contain secrets** — they're committed.

For per-developer overrides, add `src/environments/environment.local.ts` and gitignore it.

### Option B: `.env` via `@angular-builders/custom-webpack` or recent Angular CLI dotenv support

Same shape as Vite but the variable names go through `DefinePlugin` (or the new built-in support) — check the project's Angular version for the supported pattern.

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

The AdMob and RevenueCat utilities have `isTesting: true` / `initializeForTesting: true` flags. These MUST be flipped to `false` in production builds. The cleanest pattern:

```typescript
// utils/admob.ts (sketch)
import { Capacitor } from '@capacitor/core';

const ADMOB_TESTING = import.meta.env.VITE_ADMOB_TESTING === 'true';
// or, for Angular: const ADMOB_TESTING = !environment.production;

await AdMob.initialize({ initializeForTesting: ADMOB_TESTING });
```

Tying the toggle to the build mode prevents the classic mistake of shipping a `true` literal to production and getting the AdMob account flagged.

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
- [ ] Test ad IDs in dev builds; production ad IDs in release builds.
- [ ] `initializeForTesting` and `isTesting` are tied to the build mode, not hardcoded `true`.
- [ ] No secret keys (Stripe `sk_`, OpenAI / Anthropic, RevenueCat REST secret) anywhere in the bundle.
- [ ] A server-side proxy exists for any call that requires a secret.
