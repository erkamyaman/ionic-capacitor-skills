---
name: ionic-cmp-consent
description: Wire Google's User Messaging Platform (UMP) consent dialog into a Capacitor app so AdMob can serve personalized ads in EU/UK/Switzerland without the account being throttled or flagged. Trigger when adding GDPR consent, IAB TCF dialog, or "consent before AdMob" to an Ionic Capacitor app.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Google UMP Consent (GDPR / IAB TCF v2)

Without a CMP, Google AdMob serves only **non-personalized** ads to EU/UK/Switzerland traffic — and may eventually disable serving entirely for non-compliant apps. UMP is Google's free CMP and integrates directly with `@capacitor-community/admob`.

## When to consult

- **Configure UMP in AdMob console**: [setup.md](references/setup.md)
- **Show the consent form before initializing AdMob**: [consent-flow.md](references/consent-flow.md)
- **Per-framework integration snippets**: [framework-snippets.md](references/framework-snippets.md)

## Hard rules

- ✅ Run the consent flow **before** `AdMob.initialize()`. Skipping this means EU users get non-personalized ads at best, no ads at worst.
- ✅ Show a "Privacy options" entry in Settings so users can change consent later — required by IAB TCF.
- ❌ Do not show the form on every launch. Cache the consent status; only show when status is `REQUIRED`.

## Pairs with `ionic-att`

If the app ships to iOS and serves AdMob personalized ads, you also need iOS App Tracking Transparency — see [`../ionic-att/SKILL.md`](../ionic-att/SKILL.md). The required full sequence is:

```
ATT.requestPermission()       (iOS only — do this first)
  ↓
gatherConsent() (UMP)         (this skill)
  ↓
AdMob.initialize()
  ↓
showBannerAd()                (only if not premium)
```

Both flows fire **after onboarding completes**, not on first launch — Apple rejects apps that prompt for ATT before the user has a chance to understand why.

## Library

```bash
npm install @capacitor-community/admob
npx cap sync
```

`@capacitor-community/admob` ships UMP support in the same package — no separate install. See [admob.md](../ionic-shared/references/admob.md) for the base AdMob skill.
