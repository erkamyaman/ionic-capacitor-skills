# Framework Integration Snippets

## Angular (Signals)

```typescript
// services/auth.service.ts
import { Injectable, signal, Signal, computed } from '@angular/core';
import { User } from '@supabase/supabase-js';
import { supabase } from '../utils/supabase';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly _user = signal<User | null>(null);
  readonly user: Signal<User | null> = this._user.asReadonly();
  readonly isSignedIn = computed(() => this._user() !== null);

  async initialize() {
    const { data: { session } } = await supabase.auth.getSession();
    this._user.set(session?.user ?? null);

    supabase.auth.onAuthStateChange((_event, session) => {
      this._user.set(session?.user ?? null);
    });
  }

  signOut() { return supabase.auth.signOut(); }
}
```

Bootstrap in `AppComponent`:

```typescript
async ngOnInit() {
  await this.authService.initialize();
}
```

## React

```tsx
// hooks/useAuth.ts
import { useEffect, useState } from 'react';
import { User } from '@supabase/supabase-js';
import { supabase } from '../utils/supabase';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
      setLoading(false);
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setUser(session?.user ?? null);
    });

    return () => subscription.unsubscribe();
  }, []);

  return { user, isSignedIn: user !== null, loading };
}
```

## Vue

```typescript
// composables/useAuth.ts
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { User, Subscription } from '@supabase/supabase-js';
import { supabase } from '../utils/supabase';

export function useAuth() {
  const user = ref<User | null>(null);
  const loading = ref(true);
  let sub: Subscription | null = null;

  onMounted(async () => {
    const { data: { session } } = await supabase.auth.getSession();
    user.value = session?.user ?? null;
    loading.value = false;

    const { data } = supabase.auth.onAuthStateChange((_event, session) => {
      user.value = session?.user ?? null;
    });
    sub = data.subscription;
  });

  onUnmounted(() => sub?.unsubscribe());

  return {
    user,
    isSignedIn: computed(() => user.value !== null),
    loading,
  };
}
```

## Realtime cleanup

For any component using `supabase.channel(...).subscribe()`, always `removeChannel(channel)` in cleanup. Otherwise the WebSocket leaks across page navigations.

```typescript
// React
useEffect(() => {
  const channel = supabase.channel('...').on(...).subscribe();
  return () => { supabase.removeChannel(channel); };
}, []);

// Vue
onMounted(() => { channel = supabase.channel(...).on(...).subscribe(); });
onUnmounted(() => { supabase.removeChannel(channel); });

// Angular service — implements OnDestroy
ngOnDestroy() { supabase.removeChannel(this.channel); }
```
