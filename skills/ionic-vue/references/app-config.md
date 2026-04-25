# App Configuration (Vue)

## `main.ts`

```typescript
import { createApp } from 'vue';
import { IonicVue } from '@ionic/vue';
import App from './App.vue';
import router from './router';
import { createI18n } from 'vue-i18n';
import en from './assets/i18n/en.json';
import tr from './assets/i18n/tr.json';

/* Core CSS required for Ionic components */
import '@ionic/vue/css/core.css';
import '@ionic/vue/css/normalize.css';
import '@ionic/vue/css/structure.css';
import '@ionic/vue/css/typography.css';
import '@ionic/vue/css/padding.css';
import '@ionic/vue/css/float-elements.css';
import '@ionic/vue/css/text-alignment.css';
import '@ionic/vue/css/text-transformation.css';
import '@ionic/vue/css/flex-utils.css';
import '@ionic/vue/css/display.css';
import '@ionic/vue/css/palettes/dark.class.css';

import './theme/variables.css';

const browserLang = navigator.language.split('-')[0];

// Exported so other modules (e.g., a saved-language hydration step) can read/write
// the locale via `i18n.global.locale.value` — see i18n-vue-i18n.md.
export const i18n = createI18n({
  legacy: false,
  locale: ['en', 'tr'].includes(browserLang) ? browserLang : 'en',
  fallbackLocale: 'en',
  messages: { en, tr },
});

const app = createApp(App);
// `mode: 'md'` forces Material Design styling on iOS too — gives a consistent
// look across both platforms. Drop the option (or set per-platform) to let
// Ionic auto-detect (iOS users see iOS-native, Android users see Material).
app.use(IonicVue, { mode: 'md' });
app.use(router);
app.use(i18n);
router.isReady().then(() => app.mount('#app'));
```

`legacy: false` opts into the Composition API mode of `vue-i18n` — required for `<script setup>` and `useI18n()`.

## `App.vue`

```vue
<template>
  <ion-app>
    <ion-router-outlet v-if="ready" />
  </ion-app>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import { initializeAdMob } from './utils/admob';
import { initializePurchases } from './utils/purchases';
import { applyTheme, getTheme } from './utils/theme';

const ready = ref(false);

onMounted(async () => {
  // Initialize the three shared concerns in parallel.
  // initializePurchases() MUST complete before isPremiumUser() is called
  // (e.g., from TabsLayout's onMounted banner-show), otherwise the banner
  // shows briefly to premium users.
  // try/finally ensures `ready` flips even if one init throws — without it,
  // a single rejection leaves the router outlet hidden forever (blank screen).
  try {
    await Promise.all([
      getTheme().then(applyTheme),
      initializePurchases(),
      initializeAdMob(),
    ]);
  } catch (e) {
    console.error('App init failed:', e);
  } finally {
    ready.value = true;
  }
});
</script>
```

`v-if="ready"` keeps the router outlet from rendering until init completes — prevents flash of incorrect banner state and lets the onboarding `beforeEach` guard's storage read settle.

The shared utils each `init*` call uses live in `../../ionic-shared/references/`:

- `getTheme` / `applyTheme` → [theming.md](../../ionic-shared/references/theming.md)
- `initializePurchases` → [revenuecat.md](../../ionic-shared/references/revenuecat.md)
- `initializeAdMob` → [admob.md](../../ionic-shared/references/admob.md)

Onboarding state (`isOnboardingCompleted`) is read on-demand by the `beforeEach` guard — no separate `initialize()` call needed.
