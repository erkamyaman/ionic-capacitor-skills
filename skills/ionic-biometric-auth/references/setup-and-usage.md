# Setup & Usage

## Install

```bash
npm install @aparajita/capacitor-biometric-auth
npx cap sync
```

## `Info.plist` (iOS)

```xml
<key>NSFaceIDUsageDescription</key>
<string>Use Face ID to unlock the app.</string>
```

Required for Face ID. App Review rejects builds that reference Face ID without this string.

## Android `AndroidManifest.xml`

The plugin adds the `USE_BIOMETRIC` and `USE_FINGERPRINT` permissions automatically — no manual edit.

## Check availability

```typescript
import { BiometricAuth } from '@aparajita/capacitor-biometric-auth';

const info = await BiometricAuth.checkBiometry();

console.log(info.isAvailable);    // true if device has biometrics enrolled
console.log(info.biometryType);   // 'faceId' | 'touchId' | 'fingerprintAuthentication' | 'faceAuthentication' | 'irisAuthentication' | 'none'
console.log(info.reason);         // when isAvailable === false, why
```

If `isAvailable` is false, fall back to passcode / password — never block the user.

## Authenticate

```typescript
try {
  await BiometricAuth.authenticate({
    reason: 'Unlock the app',                          // shown in iOS prompt
    cancelTitle: 'Cancel',                             // iOS button
    allowDeviceCredential: true,                       // fall back to passcode/PIN
    iosFallbackTitle: 'Use Passcode',
    androidTitle: 'Unlock',
    androidSubtitle: 'Verify your identity',
    androidConfirmationRequired: false,
  });

  // Success — proceed
} catch (err: any) {
  // err.code tells you why
  // 'authenticationFailed' | 'userCancel' | 'systemCancel' | 'biometryNotEnrolled' | ...
  console.error(err.code);
}
```

`allowDeviceCredential: true` lets the user fall back to the device PIN / passcode if biometrics fail or aren't enrolled — critical for usability.

## App lock pattern

```typescript
// utils/app-lock.ts
import { BiometricAuth } from '@aparajita/capacitor-biometric-auth';
import { Preferences } from '@capacitor/preferences';

const KEY = 'appLockEnabled';

export async function isAppLockEnabled(): Promise<boolean> {
  const { value } = await Preferences.get({ key: KEY });
  return value === 'true';
}

export async function setAppLockEnabled(enabled: boolean): Promise<void> {
  await Preferences.set({ key: KEY, value: String(enabled) });
}

export async function unlock(): Promise<boolean> {
  if (!(await isAppLockEnabled())) return true;

  const { isAvailable } = await BiometricAuth.checkBiometry();
  if (!isAvailable) return true;  // no biometrics enrolled — skip

  try {
    await BiometricAuth.authenticate({
      reason: 'Unlock the app',
      allowDeviceCredential: true,
    });
    return true;
  } catch {
    return false;
  }
}
```

Then call `unlock()`:

- On app **resume** after the app has been backgrounded for **>5 minutes** (low-sensitivity apps) or **>1 minute** (banking / medical / similarly sensitive). Use `@capacitor/app`'s `appStateChange` listener to capture a `lastBackgrounded` timestamp; on the `active=true` event, compare and call `unlock()` if the threshold is crossed.
- On **deep link** into a sensitive route.
- Not on every API call.

```typescript
import { App } from '@capacitor/app';
import { Preferences } from '@capacitor/preferences';

const LOCK_AFTER_MS = 5 * 60 * 1000;   // 5 minutes — adjust per app sensitivity

App.addListener('appStateChange', async ({ isActive }) => {
  if (!isActive) {
    await Preferences.set({ key: 'lastBackgrounded', value: String(Date.now()) });
    return;
  }
  const { value } = await Preferences.get({ key: 'lastBackgrounded' });
  const last = Number(value ?? 0);
  if (Date.now() - last > LOCK_AFTER_MS) {
    const ok = await unlock();
    if (!ok) { /* navigate to a lock screen, sign out, etc. */ }
  }
});
```

## Settings page entry

```
[ ] Use Face ID to unlock          (toggle)
```

When the user enables it, immediately call `BiometricAuth.authenticate()` once to verify they can — don't wait until next launch to find out it's broken.

## What biometrics don't give you

- They don't sign requests.
- They don't encrypt anything.
- They don't authenticate you to a server.

For any of those, pair biometrics with a **server-issued token** stored after a real password / OAuth login. Biometrics are a local "yes the device owner is here" signal.
