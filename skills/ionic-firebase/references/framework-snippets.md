# Framework Integration Snippets

Where to surface Firebase Auth state and Firestore subscriptions per framework.

## Angular

Two routes — pick one:

### A. Firebase JS SDK directly + Signals

```typescript
// services/auth.service.ts
import { Injectable, signal, Signal, computed } from '@angular/core';
import { onAuthStateChanged, User } from 'firebase/auth';
import { auth } from '../utils/firebase';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly _user = signal<User | null>(null);
  readonly user: Signal<User | null> = this._user.asReadonly();
  readonly isSignedIn = computed(() => this._user() !== null);

  constructor() {
    onAuthStateChanged(auth, (u) => this._user.set(u));
  }
}
```

Then in any component template: `@if (auth.isSignedIn()) { ... }`.

### B. `@angular/fire`

```bash
npm install @angular/fire
```

Provides typed Observables and Angular DI integration. Recommended if your team is RxJS-fluent and the rest of the app already uses Observables.

```typescript
// app.config.ts
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideAuth, getAuth } from '@angular/fire/auth';
import { provideFirestore, getFirestore } from '@angular/fire/firestore';
import { firebaseConfig } from './utils/firebase';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    provideFirebaseApp(() => initializeApp(firebaseConfig)),
    provideAuth(() => getAuth()),
    provideFirestore(() => getFirestore()),
  ],
};
```

Then inject `Auth` / `Firestore` in services.

## React

```tsx
// hooks/useAuth.ts
import { useEffect, useState } from 'react';
import { onAuthStateChanged, User } from 'firebase/auth';
import { auth } from '../utils/firebase';

export function useAuth() {
  const [user, setUser] = useState<User | null>(auth.currentUser);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    return onAuthStateChanged(auth, (u) => {
      setUser(u);
      setLoading(false);
    });
  }, []);

  return { user, isSignedIn: user !== null, loading };
}
```

```tsx
// components/AuthGuard.tsx
import { useAuth } from '../hooks/useAuth';
import { Redirect } from 'react-router-dom';

export const AuthGuard: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return null;
  if (!user) return <Redirect to="/login" />;
  return <>{children}</>;
};
```

For richer needs (queries, mutations, caching), `react-firebase-hooks` is the canonical companion library.

## Vue

```typescript
// composables/useAuth.ts
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { onAuthStateChanged, User } from 'firebase/auth';
import { auth } from '../utils/firebase';

export function useAuth() {
  const user = ref<User | null>(auth.currentUser);
  const loading = ref(true);
  let unsub: (() => void) | null = null;

  onMounted(() => {
    unsub = onAuthStateChanged(auth, (u) => {
      user.value = u;
      loading.value = false;
    });
  });

  onUnmounted(() => unsub?.());

  return {
    user,
    isSignedIn: computed(() => user.value !== null),
    loading,
  };
}
```

For Vue, `vuefire` provides the equivalent of `@angular/fire` / `react-firebase-hooks` — handy for typed Firestore subscriptions.

## Auth-aware routing

The existing onboarding guard becomes a two-stage check: onboarding completed AND user signed in. If you have both, gate them in the same place — see your framework's `onboarding-guard.md` and extend the predicate.

## Subscribe → unsubscribe

Every `onSnapshot` in Firestore returns an unsubscribe function. **Always call it** in cleanup (Angular `OnDestroy`, React `useEffect` cleanup, Vue `onUnmounted`). Leaked listeners hold WebSocket connections and bill reads on every change.
