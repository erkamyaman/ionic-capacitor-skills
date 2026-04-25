# Push Notifications (`@capacitor/push-notifications`)

Permission request + listener wiring. Sending notifications themselves is done from a backend (FCM / APNs) — this scaffold only covers the client side.

## Utility (framework-agnostic)

```typescript
// utils/notifications.ts
import { Capacitor } from '@capacitor/core';
import { PushNotifications } from '@capacitor/push-notifications';

export async function requestNotificationPermission(): Promise<boolean> {
  if (!Capacitor.isNativePlatform()) return false;

  const result = await PushNotifications.requestPermissions();
  if (result.receive === 'granted') {
    await PushNotifications.register();
    return true;
  }
  return false;
}

export async function addNotificationListeners(): Promise<void> {
  await PushNotifications.addListener('registration', (token) => {
    console.log('Push registration success, token:', token.value);
  });

  await PushNotifications.addListener('registrationError', (error) => {
    console.error('Push registration error:', error);
  });

  await PushNotifications.addListener('pushNotificationReceived', (notification) => {
    console.log('Push notification received:', notification);
  });

  await PushNotifications.addListener('pushNotificationActionPerformed', (action) => {
    console.log('Push notification action:', action);
  });
}
```

## When to ask for permission

- **Not** on app launch — that's a UX anti-pattern and tanks acceptance rates.
- Prompt after the user has experienced enough of the app to understand the value (e.g., on the Settings page when they toggle notifications on, or after first meaningful action).
- The Settings page should expose an "Enable notifications" toggle that calls `requestNotificationPermission()` on enable.

## Native setup

- **iOS**: enable Push Notifications capability in Xcode (`npx cap open ios` → Signing & Capabilities). Configure APNs key in the Apple Developer portal and add it to your push provider (FCM, OneSignal, etc.).
- **Android**: requires `google-services.json` in `android/app/` and FCM enabled in Firebase. As of Android 13+, the `POST_NOTIFICATIONS` runtime permission is requested by `requestPermissions()` automatically.

## Token handling

The `registration` listener fires once with the device token. Forward that token to your backend so it can target this device.
