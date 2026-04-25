# i18n with `vue-i18n`

Configuration lives in `main.ts` (`createI18n({ legacy: false, ... })`) — see [app-config.md](app-config.md). Translation content lives in [`../../ionic-shared/references/localization-content.md`](../../ionic-shared/references/localization-content.md).

`legacy: false` is non-negotiable — the Composition API mode is what makes `useI18n()` work inside `<script setup>`.

## Usage in templates

```vue
<template>
  <h1>{{ t('settings.title') }}</h1>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
const { t } = useI18n();
</script>
```

For attribute placeholders:

```vue
<ion-input :placeholder="t('paywall.title')" />
```

## Changing the language at runtime

```typescript
const { locale } = useI18n();
locale.value = 'tr';
```

Persist the change to `@capacitor/preferences` so it survives restarts:

```typescript
import { Preferences } from '@capacitor/preferences';
await Preferences.set({ key: 'language', value: 'tr' });
```

## Honoring a saved preference at startup

The `main.ts` setup detects browser language. To restore a saved choice, hydrate the locale before mounting. The snippet below assumes `i18n` is exported from `main.ts` — change `const i18n = createI18n(...)` to `export const i18n = createI18n(...)` in [app-config.md](app-config.md):

```typescript
import { Preferences } from '@capacitor/preferences';
import { i18n } from './main';

const { value: saved } = await Preferences.get({ key: 'language' });
if (saved && ['en', 'tr'].includes(saved)) {
  i18n.global.locale.value = saved as 'en' | 'tr';
}
```

Place the hydration before `app.mount('#app')` — the cleanest spot is to wrap the existing `router.isReady().then(...)` block.

## Global vs scoped i18n

Default config uses **global** i18n (one `useI18n()` returns the global instance). If you ever need scoped translations per component, pass `useI18n({ inheritLocale: true, ... })` — but for this scaffold, global is correct.
