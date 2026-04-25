# Custom URL Scheme

A custom scheme like `myapp://` is the simplest deep-link flavor. The OS routes any URL with that scheme to your app.

## Configure the scheme

### `capacitor.config.ts`

```typescript
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.company.appname',
  appName: 'App Name',
  webDir: 'dist',
  server: { androidScheme: 'https' },
};

export default config;
```

The scheme is registered on each native side separately:

### iOS — `ios/App/App/Info.plist`

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLName</key>
    <string>com.company.appname</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### Android — `android/app/src/main/AndroidManifest.xml`

Inside the launcher `<activity>`:

```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" />
</intent-filter>
```

## Listen in the app

```typescript
// utils/deep-links.ts
import { App, URLOpenListenerEvent } from '@capacitor/app';

export function registerDeepLinkListener(handler: (url: URL) => void) {
  App.addListener('appUrlOpen', (event: URLOpenListenerEvent) => {
    handler(new URL(event.url));
  });
}

export async function getInitialDeepLink(): Promise<URL | null> {
  const { url } = await App.getLaunchUrl() ?? { url: undefined };
  return url ? new URL(url) : null;
}
```

`appUrlOpen` fires for warm opens. `getLaunchUrl()` returns the URL the app was cold-started with — call it once on bootstrap.

## Test

```bash
# iOS simulator
xcrun simctl openurl booted "myapp://auth/reset-password?token=abc123"

# Android emulator / device
adb shell am start -a android.intent.action.VIEW -d "myapp://auth/reset-password?token=abc123"
```

For routing the parsed URL into a page, see [routing.md](routing.md).
