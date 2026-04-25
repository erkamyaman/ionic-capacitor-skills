---
name: ionic-biometric-auth
description: Add Face ID / Touch ID / fingerprint authentication to an Ionic Capacitor app via @aparajita/capacitor-biometric-auth. Trigger when adding biometric login, app lock, secure reauthentication, or Face ID/Touch ID gating to a sensitive feature.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Biometric Authentication

Use the device's biometric sensor (Face ID, Touch ID, fingerprint) to authenticate the user — for app lock, sensitive feature gating, or as a step-up after a session timeout.

## When to consult

- **Setup + permissions + usage**: [setup-and-usage.md](references/setup-and-usage.md)

## Hard rules

- ✅ Biometrics **authenticate the user to the device**, not to your backend. Don't treat a successful biometric prompt as a backend session — it's a local proof of presence. Pair with a server-issued token / session.
- ✅ For "app lock", store a flag in `@capacitor/preferences` and gate route navigation on it.
- ✅ Provide a **fallback** to passcode / password when biometrics fail or are unavailable.
- ❌ Don't use biometrics to "decrypt" stored secrets directly in this plugin — for true secure-enclave-backed key storage, use a secure-storage plugin backed by the iOS Keychain / Android Keystore (e.g. `capacitor-secure-storage-plugin`, or whichever community package is currently maintained — verify on npm before adopting).
- ❌ Don't prompt for biometrics on every API call — that's both annoying and not what biometrics are for. Gate at app launch / on resume after a configurable timeout.

## Library

```bash
npm install @aparajita/capacitor-biometric-auth
npx cap sync
```

Alternative plugins: `@capgo/capacitor-native-biometric` is also widely used. Pick one and stay consistent — they have similar APIs but different defaults.
