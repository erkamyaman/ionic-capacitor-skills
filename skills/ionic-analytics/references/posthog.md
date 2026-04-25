# PostHog

JS SDK works in Capacitor WebView with no native plugin required.

## Install

```bash
npm install posthog-js
```

## Init

```typescript
// utils/analytics.ts
import posthog from 'posthog-js';
import { Capacitor } from '@capacitor/core';

export function initAnalytics() {
  posthog.init(import.meta.env.VITE_POSTHOG_KEY, {
    api_host: 'https://us.posthog.com',           // or eu.posthog.com / self-hosted URL
    capture_pageview: false,                       // we'll capture screens manually
    capture_pageleave: true,
    autocapture: false,                            // mobile autocapture is noisy — opt-in events instead
    persistence: 'localStorage',                   // see Capacitor Preferences note below
    loaded: (ph) => {
      if (import.meta.env.DEV) ph.opt_out_capturing();
    },
  });

  // Tag every event with the platform
  posthog.register({
    platform: Capacitor.getPlatform(),
    is_native: Capacitor.isNativePlatform(),
  });
}
```

The PostHog **project API key** is publishable — safe to ship.

> **Storage caveat**: PostHog defaults to `localStorage`. In Capacitor WebView this works, but cleared storage means lost identity. For a more durable setup, write a small adapter that mirrors PostHog's storage to `@capacitor/preferences` — or accept the loss for typical use cases.

## Track an event

```typescript
import posthog from 'posthog-js';

posthog.capture('paywall_viewed', {
  source: 'onboarding',
  plan_offered: 'weekly,yearly',
});
```

## Identify the user

```typescript
// After sign-in — use a stable internal user ID, not the email
posthog.identify(userId, {
  // User properties
  plan: 'yearly',
  language: 'tr',
  signup_date: '2026-04-25',
});

// Pre-sign-in (anonymous) → after sign-in (identified): events stitch automatically
// PostHog uses the previous distinct ID as `$anon_distinct_id`

// On sign out
posthog.reset();
```

## Screen views

```typescript
posthog.capture('$screen', { $screen_name: 'PaywallPage' });
```

Hook into router events the same way as Firebase Analytics — see [firebase-analytics.md](firebase-analytics.md) for Angular / React / Vue snippets; just swap the call.

## Feature flags

```typescript
// reloadFeatureFlags() is callback-based, not awaitable — the request fires
// in the background. Use posthog.onFeatureFlags() to react when results arrive.
posthog.reloadFeatureFlags();

posthog.onFeatureFlags(() => {
  // Now safe to read flags.
  if (posthog.isFeatureEnabled('new_paywall_design')) {
    // show variant
  }

  // Multivariate
  const variant = posthog.getFeatureFlag('paywall_copy');
  // 'control' | 'shorter' | 'longer' | undefined
});
```

## Session replay (web/WebView)

Session replay is **opt-in via the PostHog dashboard** — Project Settings → Session Replay → enable. Once enabled, the JS SDK records automatically. Configure recording behavior + masking at `init` time:

```typescript
posthog.init(KEY, {
  // ...
  // Recording config (no top-level `enable_session_recording` flag — that's set in dashboard).
  session_recording: {
    recordCrossOriginIframes: false,
    maskAllInputs: true,                // mask all <input> values
    maskTextSelector: '.private',       // mask any element matching this CSS selector
  },
  // To explicitly turn recording off in code (overrides the dashboard setting):
  // disable_session_recording: true,
});
```

For native iOS / Android session replay, PostHog ships dedicated SDKs (`posthog-ios` / `posthog-android`); for Capacitor apps using `posthog-js` only, recording is limited to the WebView. Check current PostHog docs before assuming feature parity.

## Privacy

```typescript
// posthog-js does NOT have a runtime startSessionRecording(...) that accepts
// masking options — masking is configured via session_recording in posthog.init()
// (see above) or via the dashboard.

// Opt the user out entirely (e.g., from a Settings toggle)
posthog.opt_out_capturing();
posthog.opt_in_capturing();
```

## Avoid double-tracking

If the app uses both Firebase (for IAP-related analytics) and PostHog (for product analytics), be deliberate about which events go where. Don't send the same event to both — pick a primary tool per event category.
