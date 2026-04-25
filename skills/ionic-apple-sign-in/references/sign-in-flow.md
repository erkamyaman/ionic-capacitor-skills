# Sign-in Flow

## Utility (framework-agnostic)

```typescript
// utils/apple-sign-in.ts
import { Capacitor } from '@capacitor/core';
import {
  SignInWithApple,
  SignInWithAppleOptions,
  SignInWithAppleResponse,
} from '@capacitor-community/apple-sign-in';

function generateNonce(): string {
  // Cryptographically random nonce — your backend will verify this matches
  // the nonce in Apple's returned JWT.
  // 16 bytes = 32 hex chars; Apple's docs recommend at least 32 chars so this
  // is the minimum-acceptable length. Bump to 32 bytes (64 hex) if you want more entropy.
  const arr = new Uint8Array(16);
  crypto.getRandomValues(arr);
  return Array.from(arr, b => b.toString(16).padStart(2, '0')).join('');
}

async function sha256Hex(input: string): Promise<string> {
  const buf = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(input));
  return Array.from(new Uint8Array(buf), b => b.toString(16).padStart(2, '0')).join('');
}

export async function signInWithApple(): Promise<{
  identityToken: string;
  nonce: string;
  state: string;
  user: string;
  email: string | null;
  givenName: string | null;
  familyName: string | null;
}> {
  const nonce = generateNonce();
  const hashedNonce = await sha256Hex(nonce);

  // Random state for CSRF protection — must be unique per request and verified on the backend.
  const state = generateNonce();   // reuse the same random-bytes-to-hex helper

  const isIos = Capacitor.getPlatform() === 'ios';
  const options: SignInWithAppleOptions = {
    clientId: isIos
      ? 'com.company.appname'                   // bundle ID on iOS
      : 'com.company.appname.web',              // Services ID on Android (configured in Apple Developer portal)
    // redirectURI is required for the Android web flow; ignored by the native iOS dialog.
    redirectURI: isIos ? undefined : 'https://api.yourapp.com/auth/apple/callback',
    scopes: 'email name',
    state,                                      // CSRF state — backend verifies this matches the in-flight request
    nonce: hashedNonce,                         // SHA-256 of the raw nonce
  };

  const response: SignInWithAppleResponse = await SignInWithApple.authorize(options);
  const { identityToken, user, email, givenName, familyName } = response.response;

  // Send the JWT + raw nonce + state to your backend for verification.
  // Backend checks: (1) JWT's nonce equals SHA-256(rawNonce); (2) state matches
  // the value the backend issued for this in-flight request (CSRF).
  return {
    identityToken,
    nonce,                  // raw nonce — backend hashes & compares to JWT
    state,                  // CSRF state — backend verifies and discards
    user,                   // stable Apple user identifier
    email,                  // null after first sign-in (see below)
    givenName,
    familyName,
  };
}
```

## First-sign-in name + email

Apple returns `email`, `givenName`, `familyName` **only the first time the user signs into your app**. On subsequent calls these fields are `null`.

This means your backend must persist them on first contact. Pattern:

```typescript
const result = await signInWithApple();

await fetch('/api/auth/apple', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    identityToken: result.identityToken,
    rawNonce: result.nonce,
    // Send name/email — backend stores them only if user is new
    email: result.email,
    givenName: result.givenName,
    familyName: result.familyName,
  }),
});
```

If the user signs out and back in later, the SDK returns `null` for those fields — your backend already has them from first contact, so it doesn't matter.

## "Hide My Email"

When users tick "Hide My Email", Apple gives you a relay address like `abc123@privaterelay.appleid.com`. Treat it like any other email — Apple forwards mail to the user's real address. The user can revoke the relay at any time from iOS Settings.

**Don't use the email as the user's primary key.** Use `result.user` — it's stable per app and immutable.

## Sign out

The plugin doesn't have a `signOut` — Apple doesn't offer one. To sign out, clear your own auth state (delete the backend session, clear `@capacitor/preferences`).

## Revocation

Users can revoke Sign in with Apple from **iOS Settings → [Your Name] → Password & Security → Apple ID logins**. Apple sends a server-to-server webhook to your backend's revocation URL (configured in the Services ID). Handle it by invalidating their session.

## Test

iOS Simulator works for the dialog flow but the JWT it returns is a sandbox token. Real verification requires a physical device or a TestFlight build.
