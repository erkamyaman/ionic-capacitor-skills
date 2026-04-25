# Firebase Auth

## Email / password

```typescript
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  sendPasswordResetEmail,
  signOut,
  onAuthStateChanged,
  User,
} from 'firebase/auth';
import { auth } from './firebase';

// Sign up
const cred = await createUserWithEmailAndPassword(auth, email, password);
console.log(cred.user.uid);

// Sign in
await signInWithEmailAndPassword(auth, email, password);

// Reset
await sendPasswordResetEmail(auth, email);

// Sign out
await signOut(auth);

// Subscribe to auth state
const unsubscribe = onAuthStateChanged(auth, (user: User | null) => {
  // user is null when signed out
});
```

## Google Sign-In (native)

The JS SDK's `signInWithPopup` doesn't work well in WKWebView / Android WebView. Use the Capacitor plugin instead:

```bash
npm install @capacitor-firebase/authentication
npx cap sync
```

```typescript
import { FirebaseAuthentication } from '@capacitor-firebase/authentication';
import { GoogleAuthProvider, signInWithCredential } from 'firebase/auth';
import { auth } from './firebase';

async function signInWithGoogle() {
  // Native dialog
  const result = await FirebaseAuthentication.signInWithGoogle();
  const idToken = result.credential?.idToken;
  if (!idToken) throw new Error('No idToken from Google');

  // Pass to Firebase JS SDK to keep onAuthStateChanged in sync
  const credential = GoogleAuthProvider.credential(idToken);
  await signInWithCredential(auth, credential);
}
```

The plugin opens the native Google sign-in sheet (much better than a webview popup), then you hand the ID token to the JS SDK so the rest of your code uses one source of truth.

### Native config

- **iOS**: add the URL scheme from `GoogleService-Info.plist` (`REVERSED_CLIENT_ID`) into `Info.plist` `CFBundleURLTypes`. The `@capacitor-firebase/authentication` README has the exact snippet — follow it.
- **Android**: the plugin uses `google-services.json`; add the SHA-1 of your debug + release keystores to the Firebase console under Project Settings → Your apps → Android app → SHA certificates.

## Sign in with Apple

Two viable approaches — pick one:

**A. Let `@capacitor-firebase/authentication` handle the entire flow** (simpler, recommended for most apps):

```typescript
import { FirebaseAuthentication } from '@capacitor-firebase/authentication';

async function signInWithApple() {
  // skipNativeAuth: false is the default — the plugin opens the native dialog,
  // generates+verifies the nonce internally, and signs into Firebase Auth in one step.
  // No manual signInWithCredential needed.
  await FirebaseAuthentication.signInWithApple();

  // Firebase JS SDK's auth state listener will fire with the new user.
}
```

**B. Use `@capacitor-community/apple-sign-in` directly, then hand the token to Firebase** (use this only if you need the standalone plugin's behavior — e.g., custom backend verification):

```typescript
import { FirebaseAuthentication } from '@capacitor-firebase/authentication';
import { OAuthProvider, signInWithCredential } from 'firebase/auth';

async function signInWithApple() {
  // skipNativeAuth: true tells the plugin "don't sign in to Firebase yourself,
  // just return the Apple credential so I can do it manually."
  // In this mode YOU pre-generate the nonce and pass it in.
  const result = await FirebaseAuthentication.signInWithApple({
    skipNativeAuth: true,
    nonce: '<your pre-generated SHA-256 nonce>',
  });
  const { idToken, nonce: rawNonce } = result.credential ?? {};
  if (!idToken) throw new Error('No Apple idToken');

  const provider = new OAuthProvider('apple.com');
  const credential = provider.credential({ idToken, rawNonce });
  await signInWithCredential(auth, credential);
}
```

Don't mix the two — option A's simplicity is wasted if you're also calling `signInWithCredential`, and option B only works with `skipNativeAuth: true`. Most apps want option A.

> **If your app already uses `@capacitor-community/apple-sign-in`** (the standalone plugin from `ionic-apple-sign-in`), prefer option B with the standalone plugin's `signInWithApple()` instead of `@capacitor-firebase/authentication`. Don't install both. See [`../../ionic-apple-sign-in/SKILL.md`](../../ionic-apple-sign-in/SKILL.md) for the alternative.

Apple Sign-In must also be enabled in:

- Apple Developer portal → App ID capabilities.
- Firebase console → Authentication → Sign-in method → Apple.
- Xcode capability (see [`../../ionic-apple-sign-in/references/setup.md`](../../ionic-apple-sign-in/references/setup.md)).

## Magic link (passwordless)

> **Firebase Dynamic Links was fully shut down on 25 Aug 2025.** Do **not** pass `dynamicLinkDomain` — the parameter no longer works. Magic links must round-trip through your own deep-link infrastructure (Universal Links + App Links via [`../../ionic-deep-links/`](../../ionic-deep-links/SKILL.md)).

The flow:

1. App calls `sendSignInLinkToEmail(...)` with `url` set to a `https://yourapp.com/...` route covered by your AASA / `assetlinks.json`.
2. User taps the link in their email.
3. iOS / Android open the app via the Universal Link / App Link.
4. Capacitor's `@capacitor/app` `appUrlOpen` listener fires with the full `https://...` URL (which contains the Firebase sign-in tokens in the query string).
5. App calls `signInWithEmailLink` with that URL.

```typescript
// 1. Send the link
import { sendSignInLinkToEmail } from 'firebase/auth';
import { Preferences } from '@capacitor/preferences';

await sendSignInLinkToEmail(auth, email, {
  // The URL must be on a domain covered by your AASA (iOS) and assetlinks.json
  // (Android) so it opens the app via Universal Link / App Link instead of the browser.
  url: 'https://yourapp.com/auth/finish-signin',
  handleCodeInApp: true,
  // Note: the `iOS`/`android` nested options (`installApp`, `minimumVersion`,
  // `bundleId`, `packageName`) were used by Firebase Dynamic Links — that
  // service was shut down on 25 Aug 2025 and the nested fields are no-ops now.
  // Don't include them.
});

// Save the email so step 5 can complete the sign-in.
await Preferences.set({ key: 'emailForSignIn', value: email });
```

```typescript
// 2-5. Handle the deep link
import { App } from '@capacitor/app';
import { isSignInWithEmailLink, signInWithEmailLink } from 'firebase/auth';
import { auth } from './firebase';
import { Preferences } from '@capacitor/preferences';

async function handleAuthLink(rawUrl: string) {
  // Firebase's helper accepts the URL string directly — pass the FULL URL,
  // including query params (oobCode, mode, lang, apiKey).
  if (!isSignInWithEmailLink(auth, rawUrl)) return;

  const { value: email } = await Preferences.get({ key: 'emailForSignIn' });
  if (!email) {
    // First open on a different device, or storage was cleared.
    // Prompt the user to re-enter their email, then call signInWithEmailLink.
    return;
  }

  await signInWithEmailLink(auth, email, rawUrl);
  await Preferences.remove({ key: 'emailForSignIn' });
}

// Cold start
const launch = await App.getLaunchUrl();
if (launch?.url) handleAuthLink(launch.url);

// Warm opens
App.addListener('appUrlOpen', (event) => handleAuthLink(event.url));
```

**Do not** read `window.location.href` — in a native Capacitor app, the deep link arrives via `appUrlOpen`, not as a top-level browser navigation. `window.location.href` will point at `capacitor://localhost/` or similar and the magic link will silently fail.

For the deep-link plumbing (Associated Domains entitlement, AASA file, `assetlinks.json`, `intent-filter`), see [`../../ionic-deep-links/`](../../ionic-deep-links/SKILL.md).
