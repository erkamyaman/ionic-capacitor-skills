# Setup

## Apple Developer portal

1. Go to <https://developer.apple.com/account/resources/identifiers>.
2. Open your App ID.
3. Enable **Sign in with Apple** capability.
4. Save.

If the app needs to share Apple ID auth between an app and a website (or multiple apps), configure a **Services ID** with a Return URL pointing to your backend's OAuth callback.

## Xcode

`npx cap open ios` → App target → **Signing & Capabilities** → **+ Capability** → **Sign in with Apple**.

This adds to `App.entitlements`:

```xml
<key>com.apple.developer.applesignin</key>
<array>
  <string>Default</string>
</array>
```

## Android (optional)

Apple Sign-In on Android uses Apple's web OAuth flow:

1. In the Apple Developer portal, configure a **Services ID** with:
   - Identifier (e.g. `com.company.appname.web`).
   - Domain and Return URL pointing to your backend (or Firebase Auth handler).
2. The plugin uses the Services ID's `clientId` and the configured Return URL on Android.

If your app is iOS-only, skip Android setup.

## Backend prerequisite

Apple returns a **JWT ID token** signed by Apple. Your backend must:

1. Verify the JWT signature against Apple's public keys (<https://appleid.apple.com/auth/keys>).
2. Verify the audience (`aud`) matches your bundle ID / Services ID.
3. Verify the issuer (`iss`) is `https://appleid.apple.com`.
4. Verify the nonce in the JWT matches the one your app sent.

Most backends already support this — Firebase Auth, Auth0, Supabase, Clerk, AWS Cognito, etc. all have built-in Apple verification. Don't roll your own JWT verification unless you have to.
