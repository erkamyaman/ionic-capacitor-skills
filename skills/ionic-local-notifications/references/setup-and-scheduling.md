# Setup & Scheduling

## Install

```bash
npm install @capacitor/local-notifications
npx cap sync
```

## Permissions

```typescript
import { LocalNotifications } from '@capacitor/local-notifications';

const status = await LocalNotifications.checkPermissions();
if (status.display === 'prompt' || status.display === 'prompt-with-rationale') {
  const result = await LocalNotifications.requestPermissions();
  if (result.display !== 'granted') return; // user denied
}
```

iOS shows the system permission dialog. Android 13+ shows the runtime `POST_NOTIFICATIONS` prompt.

## Schedule a one-shot

```typescript
await LocalNotifications.schedule({
  notifications: [
    {
      id: 1,
      title: 'Time to drink water',
      body: 'Stay hydrated 💧',
      schedule: { at: new Date(Date.now() + 60 * 60 * 1000) }, // in 1 hour
    },
  ],
});
```

## Schedule a recurring

```typescript
// Every day at 09:00 local
await LocalNotifications.schedule({
  notifications: [
    {
      id: 100,
      title: 'Morning check-in',
      body: 'How are you feeling today?',
      schedule: {
        on: { hour: 9, minute: 0 },
        allowWhileIdle: true,            // fire even in low-power mode (Android)
      },
    },
  ],
});

// Every Monday at 18:00
await LocalNotifications.schedule({
  notifications: [
    {
      id: 101,
      title: 'Weekly summary',
      body: 'Your week in review',
      schedule: {
        on: { weekday: 2, hour: 18, minute: 0 },  // 2 = Monday in Capacitor's enum
      },
    },
  ],
});
```

`weekday` is 1-based with Sunday as 1, Monday as 2, …, Saturday as 7 — see the [`@capacitor/local-notifications` docs](https://capacitorjs.com/docs/apis/local-notifications#schedule). The convention surprises Java/iOS-native devs (Java's `Calendar.SUNDAY` is also 1, but most JS conventions are 0-based with Sunday as 0).

## Cancel

```typescript
// Cancel specific
await LocalNotifications.cancel({ notifications: [{ id: 1 }, { id: 100 }] });

// Cancel all pending
const pending = await LocalNotifications.getPending();
await LocalNotifications.cancel({ notifications: pending.notifications });
```

## List pending

```typescript
const { notifications } = await LocalNotifications.getPending();
console.log(notifications);   // each has { id, title, body, schedule, ... }
```

## Listeners

```typescript
LocalNotifications.addListener('localNotificationReceived', (notification) => {
  // App was in foreground when notification fired
  console.log('received:', notification.title);
});

LocalNotifications.addListener('localNotificationActionPerformed', (event) => {
  // User tapped the notification (or an action button)
  console.log('tapped:', event.notification.id, event.actionId);
});
```

The `actionPerformed` event runs even from a cold start — register the listener early in app bootstrap to catch it.

## Badge count (iOS)

```typescript
await LocalNotifications.schedule({
  notifications: [{
    id: 1,
    title: 'Hello',
    body: 'You have 3 new items',
    schedule: { at: new Date(Date.now() + 5000) },
    iosBadge: 3,   // top-level field; `extra: { badge: 3 }` does NOT set the iOS app icon badge
  }],
});
```

The iOS app icon badge updates when the notification fires. Field names vary by `@capacitor/local-notifications` major version — verify against your installed version's typings if `iosBadge` errors.
