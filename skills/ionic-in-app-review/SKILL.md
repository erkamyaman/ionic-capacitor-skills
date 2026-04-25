---
name: ionic-in-app-review
description: Show the native App Store / Play Store rating prompt via @capacitor-community/in-app-review. Trigger when adding "rate this app" UX, the rating prompt after a positive moment (purchase complete, milestone reached), or any in-app review request flow.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# In-App Review

The native rating dialog — Apple's `SKStoreReviewController` and Google Play's In-App Review API. Both are quota-limited by the OS to prevent spam (you can call them; the OS decides whether to actually show).

## Install

```bash
npm install @capacitor-community/in-app-review
npx cap sync
```

## Usage

```typescript
import { InAppReview } from '@capacitor-community/in-app-review';

await InAppReview.requestReview();
```

That's it. The OS handles the dialog presentation, throttling, and submission.

## Hard rules

- ✅ Trigger after a **positive moment**: purchase completed, level finished, milestone hit, successful save. Never after an error.
- ✅ Wait until the user has used the app for at least a few sessions.
- ✅ The OS silently no-ops if it's been called too recently (Apple: ~3x per year per user — see [`SKStoreReviewController.requestReview` HIG](https://developer.apple.com/documentation/storekit/skstorereviewcontroller/requestreview()) for the published guideline). Don't try to detect this — just call when appropriate.
- ❌ Don't attach a custom UI ("Would you like to rate?" → yes → call requestReview). That's redundant — the system dialog already asks. And on Android the prompt may not appear at all if the quota is exhausted.
- ❌ Don't call after every session. Apple may rate-limit the entire app's review prompt globally.
- ❌ Don't bribe for ratings ("rate us 5 stars to unlock") — App Review rejects, and so does the Play Console.

## Recommended trigger pattern

```typescript
// utils/review.ts
import { Preferences } from '@capacitor/preferences';
import { InAppReview } from '@capacitor-community/in-app-review';

const SESSION_COUNT_KEY = 'reviewSessionCount';
const LAST_PROMPT_KEY = 'reviewLastPromptedAt';
const PROMPT_AFTER_SESSIONS = 5;
const PROMPT_COOLDOWN_DAYS = 90;

export async function maybeRequestReview() {
  const sessions = Number((await Preferences.get({ key: SESSION_COUNT_KEY })).value ?? 0);
  const lastPrompted = Number((await Preferences.get({ key: LAST_PROMPT_KEY })).value ?? 0);
  const now = Date.now();
  const cooldownMs = PROMPT_COOLDOWN_DAYS * 24 * 60 * 60 * 1000;

  if (sessions < PROMPT_AFTER_SESSIONS) return;
  if (now - lastPrompted < cooldownMs) return;

  await InAppReview.requestReview();
  await Preferences.set({ key: LAST_PROMPT_KEY, value: String(now) });
}

export async function bumpSessionCount() {
  const current = Number((await Preferences.get({ key: SESSION_COUNT_KEY })).value ?? 0);
  await Preferences.set({ key: SESSION_COUNT_KEY, value: String(current + 1) });
}
```

Then call:

- `bumpSessionCount()` once on app launch.
- `maybeRequestReview()` after a positive moment (e.g., after a successful purchase, after the user completes onboarding flow, etc.).

## You can't see who reviewed

Apple/Google return no signal about whether the user actually rated. Don't try to track conversion — track the call, not the result.

## Fall-back to deep link

Some teams add a "Rate the app" Settings button that opens the App Store / Play Store listing directly (skipping the in-app dialog) — useful for users who want to leave a written review:

```typescript
import { Browser } from '@capacitor/browser';
import { Capacitor } from '@capacitor/core';

async function openStoreListing() {
  const url = Capacitor.getPlatform() === 'ios'
    ? 'itms-apps://itunes.apple.com/app/idAPPSTORE_ID?action=write-review'
    : 'market://details?id=com.company.appname';
  await Browser.open({ url });
}
```

Replace `APPSTORE_ID` and the bundle ID with your actual values.
