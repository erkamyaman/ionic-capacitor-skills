# Android App Links

App Links are the Android counterpart to Universal Links — `https://yourapp.com/...` opens the app directly, verified by `assetlinks.json`.

## App side

### `AndroidManifest.xml`

Inside the launcher `<activity>`, add an `intent-filter` with `android:autoVerify="true"`:

```xml
<intent-filter android:autoVerify="true">
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="https" android:host="yourapp.com" />
  <data android:scheme="https" android:host="staging.yourapp.com" />
</intent-filter>
```

> **Per-host verification.** With `autoVerify="true"` and multiple `android:host` entries in one filter, Android verifies each host independently. If `staging.yourapp.com` doesn't host a valid `assetlinks.json`, the **whole filter** is marked unverified — including the production host. For dev/staging coexistence, either keep them in separate filters or skip `autoVerify` for non-prod hosts.

`autoVerify="true"` tells Android to fetch `assetlinks.json` and check it on first install.

For specific paths only:

```xml
<data
  android:scheme="https"
  android:host="yourapp.com"
  android:pathPrefix="/reset-password" />
```

## Domain side

### Get your app's signing fingerprint

For local debug builds:

```bash
keytool -list -v -keystore ~/.android/debug.keystore \
  -alias androiddebugkey -storepass android -keypass android
# Look for the SHA256 fingerprint
```

For release builds, use your release keystore. If using **Play App Signing**, get the SHA-256 from Play Console → **Setup** → **App integrity** → **App signing key certificate**.

> **Each developer's debug keystore has a unique SHA-256.** The `~/.android/debug.keystore` is auto-generated per machine — your laptop's fingerprint isn't your colleague's. Either include every team member's debug fingerprint in `assetlinks.json` (small teams), or share a single committed `debug.keystore` in the repo for App Links testing (use a separate one if you commit it — never the default keystore from your `~/.android/`). For most teams, putting only the **release / Play App Signing** fingerprint in production and accepting that App Links don't verify on local debug builds is the cleanest path.

Include all signing fingerprints you actually use (release + any debug ones you want to verify) so the corresponding builds work.

### Host `assetlinks.json`

```
https://yourapp.com/.well-known/assetlinks.json
```

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.company.appname",
      "sha256_cert_fingerprints": [
        "AB:CD:EF:01:23:45:67:89:AB:CD:EF:01:23:45:67:89:AB:CD:EF:01:23:45:67:89:AB:CD:EF:01:23:45:67:89",
        "12:34:..."
      ]
    }
  }
]
```

Add **all** fingerprints you'll use (debug, release, Play App Signing). Multiple apps? Use multiple objects in the array.

### Verify the file

```bash
curl -I https://yourapp.com/.well-known/assetlinks.json
# Look for: HTTP/2 200, Content-Type: application/json
```

Use Google's [Statement List Generator and Tester](https://developers.google.com/digital-asset-links/tools/generator) to validate.

## Test

```bash
# Install the app on emulator/device, then
adb shell am start -W -a android.intent.action.VIEW \
  -d "https://yourapp.com/reset-password?token=abc"
```

If verified, the app opens. If not, the URL opens in the browser.

Check verification status:

```bash
adb shell pm get-app-links com.company.appname
# verified | 1024 -> verified hosts
```

## What if a user has the app uninstalled?

App Links fall back to the browser (or the user's chosen handler). Make sure your web URL renders something useful.
