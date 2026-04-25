---
name: ionic-apple-sign-in
description: Add "Sign in with Apple" to an Ionic Capacitor app via @capacitor-community/apple-sign-in. Trigger when adding social login on iOS, integrating Apple ID as a sign-in method, or when App Review requires Sign in with Apple alongside other social logins.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Sign in with Apple

Apple requires apps that offer **any** other social login (Google / Facebook / etc.) on iOS to also offer **Sign in with Apple**. Ship Google sign-in without it and App Review will reject the build.

> **Don't use this skill in a Firebase project.** If your app already uses Firebase Auth, integrate Sign in with Apple via [`@capacitor-firebase/authentication`](../ionic-firebase/references/auth.md#sign-in-with-apple) instead. That plugin uses the same native dialog on iOS and the same Apple OAuth flow on Android, but hands the JWT directly to Firebase Auth so you don't end up running two different identity layers. Installing **both** `@capacitor-community/apple-sign-in` and `@capacitor-firebase/authentication` is wasteful and easy to misconfigure.
>
> Use this skill when:
> - You're not on Firebase (e.g., Supabase — see [`../ionic-supabase/references/auth.md#native-apple-sign-in`](../ionic-supabase/references/auth.md), or your own backend).
> - You verify the Apple JWT yourself on a custom backend.

## When to consult

- **App ID setup + entitlement**: [setup.md](references/setup.md)
- **Sign-in flow + nonce verification**: [sign-in-flow.md](references/sign-in-flow.md)

## Hard rules

- ✅ Required if you offer Google / Facebook / Twitter / OAuth login on iOS.
- ✅ The Apple-issued user identifier is opaque and stable per app — store it as the user's primary key.
- ✅ Apple returns the user's name and email **only on the first sign-in**. If your backend doesn't capture them then, you don't get a second chance unless the user revokes and re-grants in iOS Settings → Apple ID → Sign in with Apple.
- ✅ Generate and verify a **nonce** to prevent replay attacks.
- ❌ Don't store the Apple email as the user identifier — Apple's "Hide My Email" returns a relay address that the user can change.
- ❌ Android side: this plugin uses Apple's web flow (popup) on Android — works, but it's an OAuth web redirect, not a native dialog.

## Library

```bash
npm install @capacitor-community/apple-sign-in
npx cap sync
```
