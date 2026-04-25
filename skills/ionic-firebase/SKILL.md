---
name: ionic-firebase
description: Wire Firebase Auth (email/password, Google, Apple) and Cloud Firestore into an Ionic Capacitor app. Trigger when adding Firebase Auth, Firestore, social sign-in via Firebase, or backend persistence using Google's Firebase platform.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Firebase (Auth + Firestore)

The default backend for most Ionic apps. Auth handles sign-up / sign-in (including social providers); Firestore handles persistent data with realtime sync and offline support.

## When to consult

- **Project setup + native config files** (`google-services.json`, `GoogleService-Info.plist`): [setup.md](references/setup.md)
- **Auth** (email, Google, Apple, magic link): [auth.md](references/auth.md)
- **Firestore** (read/write/realtime/offline): [firestore.md](references/firestore.md)
- **Per-framework integration snippets**: [framework-snippets.md](references/framework-snippets.md)

## Library choice

Two viable approaches:

| Approach | When to pick |
|----------|--------------|
| **Firebase JS SDK** (`firebase` npm package) | Web-first, simple cases, web/mobile parity. Works in browser, iOS WKWebView, Android WebView. Auth popups can be tricky on native. |
| **Capacitor Firebase plugins** (`@capacitor-firebase/*`) | Native auth dialogs (Google sign-in via native sheets), offline Firestore via native SDKs, push via native FCM. More setup, much better UX. |

**Recommendation**: use the JS SDK for v1 (faster), migrate to `@capacitor-firebase/*` plugins when you hit native UX gaps (especially Google Sign-In, which is rough via web-only on Android).

This skill covers both — pick whichever fits.

## Hard rules

- ✅ **Never put Firebase Admin SDK code in the app.** That's server-only — uses a service account with full permissions.
- ✅ Firebase **web config** (`apiKey`, `projectId`, etc.) is safe to ship — Security Rules do the gating. See [`../ionic-shared/references/environments-and-keys.md`](../ionic-shared/references/environments-and-keys.md).
- ✅ Write Firestore Security Rules **before** going live. The default "test mode" rules expire after 30 days and become "deny all" if not replaced.
- ❌ Don't use Realtime Database for new projects unless you have a specific reason — Firestore is the modern choice.
- ❌ Don't query a collection with no index expecting it to scale — Firestore's query patterns require composite indexes for compound filters.

## Library

```bash
npm install firebase
# OR for native plugins:
npm install @capacitor-firebase/app @capacitor-firebase/authentication @capacitor-firebase/firestore
npx cap sync
```

## Pairs with

- **Push notifications via FCM**: Firebase's `google-services.json` / `GoogleService-Info.plist` also covers FCM. Wire push handling itself via [`../ionic-shared/references/push-notifications.md`](../ionic-shared/references/push-notifications.md) — the plugin (`@capacitor/push-notifications`) is independent of Firebase Auth/Firestore but uses the same native config files.
- **Magic-link sign-in**: requires Universal Link / App Link plumbing — see [`../ionic-deep-links/`](../ionic-deep-links/SKILL.md). Firebase Dynamic Links is shut down — your own deep-link infra is the path.
- **Sign in with Apple**: integrates via `@capacitor-firebase/authentication` (this skill, [auth.md](references/auth.md)). Don't also install `@capacitor-community/apple-sign-in` — see [`../ionic-apple-sign-in/SKILL.md`](../ionic-apple-sign-in/SKILL.md) for when to use the standalone plugin instead.
