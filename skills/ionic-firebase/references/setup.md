# Firebase Setup

## Console

1. <https://console.firebase.google.com> → **Add project**.
2. Add an **iOS app** (bundle ID matches `capacitor.config.ts` `appId`). Download `GoogleService-Info.plist`.
3. Add an **Android app** (package name = same `appId`). Download `google-services.json`.
4. Add a **Web app** to get the JS SDK config snippet (used even in Capacitor — the JS SDK runs in the WebView).

## Native config files

Place the downloaded files:

```
ios/App/App/GoogleService-Info.plist
android/app/google-services.json
```

Both must be added **inside** the native project — the JS SDK alone does not need them, but FCM, native Auth plugins, and Crashlytics do.

For iOS, after dropping the file, `npx cap open ios` and verify it appears in the App target's file list (Xcode usually picks it up automatically; if not, drag it into the project navigator).

For Android, the `google-services.json` is read by the Google Services Gradle plugin. Make sure `android/build.gradle` includes:

```groovy
buildscript {
  dependencies {
    classpath 'com.google.gms:google-services:4.4.2'
  }
}
```

And `android/app/build.gradle` ends with:

```groovy
apply plugin: 'com.google.gms.google-services'
```

The `@capacitor-firebase/*` plugins add this automatically; if you only use the JS SDK and don't install any native Firebase plugin, you can skip the gradle plumbing.

## JS SDK init

```typescript
// utils/firebase.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
```

These config values are publishable — fine to ship in the bundle. Security Rules do the gating, not the API key.

## Firestore Security Rules — required before launch

Default "test mode" rules:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // Firebase auto-fills the date as +30 days from project creation.
      allow read, write: if request.time < timestamp.date(<YYYY>, <MM>, <DD>);
    }
  }
}
```

Open at first, then **deny all** after the date passes. Replace before launch:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read/write only their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    // Users can read all posts but only write their own
    match /posts/{postId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.resource.data.authorId == request.auth.uid;
      allow update, delete: if request.auth != null && resource.data.authorId == request.auth.uid;
    }
  }
}
```

Test rules in the console's **Rules Playground** before deploying.

## Emulator suite (recommended for local dev)

```bash
npm install -g firebase-tools
firebase login
firebase init emulators
firebase emulators:start
```

In code:

```typescript
import { connectAuthEmulator } from 'firebase/auth';
import { connectFirestoreEmulator } from 'firebase/firestore';

if (import.meta.env.DEV) {
  connectAuthEmulator(auth, 'http://localhost:9099');
  connectFirestoreEmulator(db, 'localhost', 8080);
}
```

Cuts iteration time and avoids polluting prod data during dev.
