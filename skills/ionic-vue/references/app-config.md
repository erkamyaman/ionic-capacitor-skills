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

const i18n = createI18n({
  legacy: false,
  locale: ['en', 'tr'].includes(browserLang) ? browserLang : 'en',
  fallbackLocale: 'en',
  messages: { en, tr },
});

const app = createApp(App);
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
    <ion-router-outlet />
  </ion-app>
</template>

<script setup lang="ts">
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import { onMounted } from 'vue';
import { initializeAdMob } from './utils/admob';

onMounted(async () => {
  await initializeAdMob();
});
</script>
```

`initializeAdMob()` is the framework-agnostic util from `ionic-shared` — see [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md).
