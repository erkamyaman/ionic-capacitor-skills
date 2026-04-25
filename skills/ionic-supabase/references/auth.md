# Supabase Auth

## Email / password

```typescript
import { supabase } from '../utils/supabase';

// Sign up — sends a confirmation email by default
const { data, error } = await supabase.auth.signUp({ email, password });

// Sign in
await supabase.auth.signInWithPassword({ email, password });

// Sign out
await supabase.auth.signOut();

// Current session
const { data: { session } } = await supabase.auth.getSession();
const { data: { user } } = await supabase.auth.getUser();

// Subscribe to changes
const { data: { subscription } } = supabase.auth.onAuthStateChange((event, session) => {
  // event: 'INITIAL_SESSION' | 'SIGNED_IN' | 'SIGNED_OUT' | 'TOKEN_REFRESHED' | 'USER_UPDATED' | 'PASSWORD_RECOVERY'
});

// Cleanup
subscription.unsubscribe();
```

## Magic link (passwordless)

```typescript
await supabase.auth.signInWithOtp({
  email,
  options: {
    emailRedirectTo: 'myapp://auth/callback',  // your custom URL scheme
    shouldCreateUser: true,
  },
});
```

The user clicks the email link → opens the app via custom scheme → completes sign-in. Wire the deep-link handler:

```typescript
// utils/auth-callback.ts
import { App } from '@capacitor/app';
import { supabase } from './supabase';

App.addListener('appUrlOpen', async (event) => {
  const url = new URL(event.url);

  // PKCE flow (default in supabase-js v2.45+ for signInWithOAuth):
  //   redirectTo URL receives `?code=...` query param.
  //   Exchange the code for a session.
  const code = url.searchParams.get('code');
  if (code) {
    await supabase.auth.exchangeCodeForSession(url.toString());
    return;
  }

  // Implicit-grant fallback (older flow / magic links):
  //   redirectTo URL receives tokens in the hash fragment.
  if (url.hash) {
    const params = new URLSearchParams(url.hash.substring(1));
    const accessToken = params.get('access_token');
    const refreshToken = params.get('refresh_token');
    if (accessToken && refreshToken) {
      await supabase.auth.setSession({
        access_token: accessToken,
        refresh_token: refreshToken,
      });
    }
  }
});
```

Which flow your app uses depends on the `flowType` set when creating the Supabase client (`pkce` is the default in recent versions). Handling both branches keeps the code working across upgrades.

For production-grade deep linking, configure Universal Links / App Links — see [`../../ionic-deep-links/`](../../ionic-deep-links/SKILL.md).

## OAuth (Google, Apple, GitHub, …)

Web flow doesn't return cleanly to a native app. Use `skipBrowserRedirect: true` + open the URL via Capacitor Browser:

```bash
npm install @capacitor/browser
npx cap sync
```

```typescript
import { Browser } from '@capacitor/browser';
import { supabase } from '../utils/supabase';

async function signInWithGoogle() {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: 'myapp://auth/callback',
      skipBrowserRedirect: true,
    },
  });
  if (data.url) await Browser.open({ url: data.url });
}
```

The user signs in with Google in an in-app browser, gets redirected to `myapp://auth/callback` with tokens in the hash. The deep-link handler above completes the session.

In Supabase dashboard → Authentication → URL Configuration, add your custom scheme (`myapp://`) AND your Universal Link origins as **Redirect URLs**.

## Native Apple Sign-In

For better iOS UX, pair with [`ionic-apple-sign-in`](../../ionic-apple-sign-in/SKILL.md):

```typescript
import { signInWithApple as nativeAppleSignIn } from '../utils/apple-sign-in';
import { supabase } from '../utils/supabase';

async function signInWithApple() {
  const { identityToken, nonce } = await nativeAppleSignIn();

  await supabase.auth.signInWithIdToken({
    provider: 'apple',
    token: identityToken,
    nonce,                    // raw nonce — Supabase verifies the JWT's hashed nonce against this
  });
}
```

This keeps the native Apple dialog on iOS while still using Supabase's identity layer.

## Native Google Sign-In

Similar pattern with `@codetrix-studio/capacitor-google-auth`. Must be initialized once before first use, with the OAuth web client ID configured in Google Cloud Console.

```bash
npm install @codetrix-studio/capacitor-google-auth
npx cap sync
```

`capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  // ...
  plugins: {
    GoogleAuth: {
      scopes: ['profile', 'email'],
      // The web client ID from Google Cloud Console (NOT the iOS / Android client IDs).
      // Used by Supabase to verify the token's audience.
      serverClientId: 'YOUR_WEB_CLIENT_ID.apps.googleusercontent.com',
      forceCodeForRefreshToken: true,
    },
  },
};
```

```typescript
import { GoogleAuth } from '@codetrix-studio/capacitor-google-auth';
import { supabase } from '../utils/supabase';

// Once on app boot
await GoogleAuth.initialize();

// On the sign-in button
async function signInWithGoogle() {
  const result = await GoogleAuth.signIn();
  await supabase.auth.signInWithIdToken({
    provider: 'google',
    token: result.authentication.idToken,
  });
}
```

The plugin's iOS/Android setup steps (URL scheme in `Info.plist`, web client ID in `strings.xml`) are in the plugin's own README — follow it before the first build, or `signIn()` throws.

## Reset password

```typescript
await supabase.auth.resetPasswordForEmail(email, {
  redirectTo: 'myapp://auth/reset',
});

// In the deep-link handler, after the user lands on the reset page:
await supabase.auth.updateUser({ password: newPassword });
```
