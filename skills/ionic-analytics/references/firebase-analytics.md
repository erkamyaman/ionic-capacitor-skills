# Firebase Analytics

Native event tracking via `@capacitor-firebase/analytics`. Requires the same `GoogleService-Info.plist` / `google-services.json` as the rest of Firebase — see [`../../ionic-firebase/references/setup.md`](../../ionic-firebase/references/setup.md).

## Install

```bash
npm install @capacitor-firebase/app @capacitor-firebase/analytics
npx cap sync
```

## Track an event

```typescript
import { FirebaseAnalytics } from '@capacitor-firebase/analytics';

await FirebaseAnalytics.logEvent({
  name: 'paywall_viewed',
  params: {
    source: 'onboarding',
    plan_offered: 'weekly,yearly',
  },
});
```

## Identify the user

```typescript
// After sign-in
await FirebaseAnalytics.setUserId({ userId });

// User properties (stable attributes — premium tier, signup date, etc.)
await FirebaseAnalytics.setUserProperty({ key: 'plan', value: 'yearly' });
await FirebaseAnalytics.setUserProperty({ key: 'language', value: 'tr' });

// Sign out — pass undefined to clear the user ID; the plugin's TS types
// don't accept `null` (use `undefined` even though it's spelled "null"
// in older docs).
await FirebaseAnalytics.setUserId({ userId: undefined });
```

## Screen views

```typescript
await FirebaseAnalytics.setCurrentScreen({
  screenName: 'PaywallPage',
  screenClassOverride: 'Paywall',
});
```

Wire this into the framework's router for automatic page-view tracking:

### Angular

```typescript
// services/screen-tracking.service.ts
import { inject, Injectable } from '@angular/core';
import { Router, NavigationEnd } from '@angular/router';
import { filter } from 'rxjs/operators';
import { FirebaseAnalytics } from '@capacitor-firebase/analytics';

@Injectable({ providedIn: 'root' })
export class ScreenTrackingService {
  private router = inject(Router);

  start() {
    this.router.events.pipe(filter(e => e instanceof NavigationEnd)).subscribe((e) => {
      FirebaseAnalytics.setCurrentScreen({
        screenName: (e as NavigationEnd).urlAfterRedirects,
      });
    });
  }
}
```

### React

```tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { FirebaseAnalytics } from '@capacitor-firebase/analytics';

function ScreenTracker() {
  const location = useLocation();
  useEffect(() => {
    FirebaseAnalytics.setCurrentScreen({ screenName: location.pathname });
  }, [location.pathname]);
  return null;
}
```

Mount inside `<IonReactRouter>`.

### Vue

```typescript
// In router/index.ts
import { FirebaseAnalytics } from '@capacitor-firebase/analytics';

router.afterEach((to) => {
  FirebaseAnalytics.setCurrentScreen({ screenName: to.path });
});
```

## Toggle in dev

To avoid polluting prod analytics with dev data:

```typescript
import { FirebaseAnalytics } from '@capacitor-firebase/analytics';

if (import.meta.env.DEV) {
  await FirebaseAnalytics.setEnabled({ enabled: false });
}
```

## Reserved / automatic events

Firebase auto-collects:

- `first_open` — first launch
- `app_update` — version upgrade
- `screen_view` — when you call `setCurrentScreen`
- `in_app_purchase` — IAP via Apple/Google billing
- `session_start` — every session

Don't define your own events with these names.
