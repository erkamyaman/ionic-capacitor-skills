# When to Prompt for ATT

## The required order

```
First launch
  ↓
Onboarding completes
  ↓
(Optional) brief explanatory screen — "We show ads to keep the app free…"
  ↓
ATT.requestPermission()
  ↓
gatherConsent()  (UMP — see ionic-cmp-consent)
  ↓
AdMob.initialize()
```

ATT first, then UMP, then AdMob — because UMP's consent options depend on ATT status.

## Utility (framework-agnostic)

```typescript
// utils/att.ts
import { Capacitor } from '@capacitor/core';
import { AppTrackingTransparency } from '@capacitor-community/app-tracking-transparency';

// 'unsupported' is our own sentinel for non-iOS platforms — the plugin only
// returns the four iOS values. We synthesize 'unsupported' so callers can
// branch uniformly.
export type AttStatus = 'authorized' | 'denied' | 'restricted' | 'notDetermined' | 'unsupported';

export async function getAttStatus(): Promise<AttStatus> {
  if (Capacitor.getPlatform() !== 'ios') return 'unsupported';
  const { status } = await AppTrackingTransparency.getStatus();
  return status as AttStatus;
}

export async function requestAttPermission(): Promise<AttStatus> {
  if (Capacitor.getPlatform() !== 'ios') return 'unsupported';
  const current = await getAttStatus();
  if (current !== 'notDetermined') return current; // can only ask once

  const { status } = await AppTrackingTransparency.requestPermission();
  return status as AttStatus;
}
```

## You only get to ask once

`requestPermission()` shows the dialog **only** when status is `notDetermined`. After the user has chosen:

- `authorized` → personalized ads work.
- `denied` → ads serve, but non-personalized only.
- `restricted` → parental controls block tracking.

To change their mind, the user must go to **iOS Settings → Privacy → Tracking → [Your App]**. The app cannot re-prompt.

This is why pre-prompt timing matters — show an explainer screen first if the value isn't obvious. Common pattern:

```
[Custom screen]                       [System dialog]
"We show ads to keep              ->   "Allow [App] to track..."
the app free. Please              ->    [Ask App Not To Track]
allow tracking on the             ->    [Allow]
next screen for the
best experience."

[Continue button ->]
```

## Status-aware behavior

```typescript
import { AdMob } from '@capacitor-community/admob';

const status = await requestAttPermission();
// status drives whether AdMob serves personalized vs non-personalized ads.
// AdMob reads the IDFA permission state from the OS — there's no flag to pass.

await AdMob.initialize({
  initializeForTesting: import.meta.env.DEV,
});
```

The `@capacitor-community/admob` plugin does not request ATT itself, so there is no double-prompt risk and no AdMob option to disable that behavior. Just call `requestAttPermission()` first (so the IDFA state is set when AdMob initializes) and `AdMob.initialize()` second.

Note: when ATT is denied or unavailable, AdMob automatically falls back to non-personalized ads for that user — no extra code needed.

## Persisting the decision

Save the result to `@capacitor/preferences` so you don't re-call `getStatus()` on every launch:

```typescript
import { Preferences } from '@capacitor/preferences';

async function rememberAtt(status: AttStatus) {
  await Preferences.set({ key: 'attStatus', value: status });
}
```
