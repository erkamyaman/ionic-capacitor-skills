# Routing a Deep Link

Once the URL is in hand, route to the appropriate page. The handler runs **outside** the page lifecycle so it must use the framework's router programmatically.

## Where to register

Register early — in the app bootstrap — so links received before any page loads are still handled.

## Angular

```typescript
// app.component.ts
import { Component, OnInit, inject } from '@angular/core';
import { Router } from '@angular/router';
import { App } from '@capacitor/app';

@Component({ /* ... */ })
export class AppComponent implements OnInit {
  private router = inject(Router);

  async ngOnInit() {
    // Cold-start URL
    const launch = await App.getLaunchUrl();
    if (launch?.url) this.handle(launch.url);

    // Warm opens
    App.addListener('appUrlOpen', (event) => this.handle(event.url));
  }

  private handle(rawUrl: string) {
    const url = new URL(rawUrl);
    // Map external URL → in-app route
    if (url.pathname.startsWith('/auth/reset-password')) {
      const token = url.searchParams.get('token');
      this.router.navigateByUrl(`/auth/reset?token=${token}`);
    }
    // ...other patterns
  }
}
```

## React

```tsx
// App.tsx
import { useEffect } from 'react';
import { useIonRouter } from '@ionic/react';
import { App as CapacitorApp, type PluginListenerHandle } from '@capacitor/app';

const App: React.FC = () => {
  const router = useIonRouter();

  useEffect(() => {
    const handle = (rawUrl: string) => {
      const url = new URL(rawUrl);
      if (url.pathname.startsWith('/auth/reset-password')) {
        const token = url.searchParams.get('token');
        router.push(`/auth/reset?token=${token}`, 'forward');
      }
    };

    // Capture the listener handle so cleanup can await it. If the effect
    // unmounts before addListener resolves, the cleanup function still
    // runs first — track that with a `cancelled` flag so we don't leak.
    let cancelled = false;
    let handle$: Promise<PluginListenerHandle> | null = null;

    (async () => {
      const launch = await CapacitorApp.getLaunchUrl();
      if (launch?.url && !cancelled) handle(launch.url);
    })();

    handle$ = CapacitorApp.addListener('appUrlOpen', (e) => handle(e.url));

    return () => {
      cancelled = true;
      handle$?.then((h) => h.remove());
    };
  }, [router]);

  // ...
};
```

`useIonRouter()` is unavailable outside `IonReactRouter`, so this code lives **inside** `<IonReactRouter>`.

## Vue

```vue
<!-- App.vue -->
<script setup lang="ts">
import { onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { App as CapacitorApp } from '@capacitor/app';

const router = useRouter();

function handle(rawUrl: string) {
  const url = new URL(rawUrl);
  if (url.pathname.startsWith('/auth/reset-password')) {
    const token = url.searchParams.get('token');
    router.push(`/auth/reset?token=${token}`);
  }
}

onMounted(async () => {
  const launch = await CapacitorApp.getLaunchUrl();
  if (launch?.url) handle(launch.url);

  CapacitorApp.addListener('appUrlOpen', (e) => handle(e.url));
});
</script>
```

## Patterns

- **Always parse the URL with `new URL(...)`**, not regex. Handles encoded chars and edge cases correctly.
- **Map paths explicitly** — don't blindly forward the path; an external URL `/admin/secrets` shouldn't necessarily map to an internal `/admin/secrets`.
- **Validate query params** before using them (`token` should be expected length/format).
- **Mind cold-start race conditions** — if your app needs auth state to render, the deep-link handler may need to wait for that state before navigating.
