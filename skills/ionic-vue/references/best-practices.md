# Vue Best Practices

## Composition API + `<script setup>` only

❌ Options API:

```vue
<script>
export default {
  data() {
    return { title: 'Home' };
  },
};
</script>
```

✅ Composition API + `<script setup lang="ts">` + `<ion-page>`:

```vue
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>{{ t('home.title') }}</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <!-- content -->
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
} from '@ionic/vue';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
</script>
```

## Always wrap pages in `<ion-page>`

This is the most common Vue-Ionic mistake. `<ion-page>` is what enables Ionic's page transitions and lifecycle. Without it, transitions break and the page may not render correctly inside a router outlet.

## Navigation

Use `useRouter()` from `vue-router`:

```typescript
import { useRouter } from 'vue-router';

const router = useRouter();
router.push('/settings');
router.replace('/onboarding');   // for guard redirects (no back-button return)
```

`router.replace()` is the equivalent of Angular's `replaceUrl: true` and React's `'replace'` action.

## Icons

Import icons from `ionicons/icons` and pass as **bound values**:

```vue
<script setup lang="ts">
import { home } from 'ionicons/icons';
</script>

<template>
  <ion-icon :icon="home" />     <!-- ✅ -->
  <ion-icon name="home" />      <!-- ❌ Angular pattern -->
</template>
```

## TypeScript

- `strict: true`, no `any`.
- `<script setup lang="ts">` for every component.
- Type Capacitor plugin responses explicitly.
- Use `defineProps<{...}>()` for typed props (Composition API generic-arg form).

## State management

- For local component state, `ref()` and `reactive()`.
- For shared state across views, **composables** (functions that return `ref`s / methods).
- Reach for **Pinia** only when the app genuinely needs a centralized store with devtools / persistence — for the surface this scaffold covers, composables are enough.
