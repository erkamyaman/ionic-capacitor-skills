# Patterns

## Action buttons

Add tappable buttons that appear when the user expands the notification:

```typescript
await LocalNotifications.registerActionTypes({
  types: [
    {
      id: 'WATER_CHECKIN',
      actions: [
        { id: 'YES', title: 'Done ✓' },
        { id: 'NO',  title: 'Skip',  destructive: true },
      ],
    },
  ],
});

await LocalNotifications.schedule({
  notifications: [{
    id: 1,
    title: 'Did you drink water?',
    body: 'Tap to log it',
    actionTypeId: 'WATER_CHECKIN',
    schedule: { at: new Date(Date.now() + 60_000) },
  }],
});

LocalNotifications.addListener('localNotificationActionPerformed', (event) => {
  if (event.actionId === 'YES') logHydration();
  if (event.actionId === 'NO')  skipReminder();
});
```

## Android channels

Android groups notifications by channel — each channel has its own importance, sound, vibration. Set channels up at app start:

```typescript
import { Capacitor } from '@capacitor/core';

if (Capacitor.getPlatform() === 'android') {
  await LocalNotifications.createChannel({
    id: 'reminders',
    name: 'Daily reminders',
    description: 'Hydration and check-in nudges',
    importance: 4,             // 1-5; 4 = High (heads-up)
    sound: 'default',
    vibration: true,
    lights: true,
  });
}

await LocalNotifications.schedule({
  notifications: [{
    id: 1,
    title: '...',
    body: '...',
    channelId: 'reminders',
    schedule: { at: ... },
  }],
});
```

Without a channel, Android 8+ shows the notification but the user can't customize per-category.

## Settings page integration

The Settings page from the core scaffold has a Notifications toggle. For local-notification features, expand it:

```
Notifications
  ├── Push notifications     [toggle]   → @capacitor/push-notifications
  ├── Daily reminder         [toggle]   → schedule/cancel local notification
  └── Reminder time          09:00      → reschedule with new time
```

Each toggle should call `requestPermissions()` if needed and either schedule or cancel the relevant notification.

## Avoid the 64-notification cap (iOS)

iOS only keeps ~64 pending notifications per app. If you schedule a year of daily reminders up front, iOS drops most of them. Pattern instead:

1. Schedule the next ~30 days only.
2. On app launch, top up: cancel past notifications, schedule the next 30 days from today.

```typescript
async function refreshDailyReminders() {
  await LocalNotifications.cancel({ notifications: oldIds.map(id => ({ id })) });

  const notifications = [];
  for (let i = 0; i < 30; i++) {
    const date = new Date();
    date.setDate(date.getDate() + i);
    date.setHours(9, 0, 0, 0);
    notifications.push({
      id: 1000 + i,
      title: 'Daily reminder',
      body: '...',
      schedule: { at: date },
    });
  }
  await LocalNotifications.schedule({ notifications });
}
```

## Don't fire while the user is using the app

iOS shows notifications even when the app is foregrounded — usually you don't want this. Either:

- Skip the notification while foreground via `App.addListener('appStateChange', ...)`.
- Or set `presentationOptions: []` in the listener config (suppresses banner when foregrounded).

## Test

In the simulator, fire short scheduled times (~5 seconds out) to see them quickly. For recurring tests, change the device clock forward.
