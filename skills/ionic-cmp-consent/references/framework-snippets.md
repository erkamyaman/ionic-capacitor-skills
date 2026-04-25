# Framework Integration Snippets

Where to call `gatherConsent()` and how to expose the Privacy Options entry per framework.

## Angular

```typescript
// app.component.ts
import { Component, OnInit, inject } from '@angular/core';
import { gatherConsent } from './utils/consent';
import { AdsService } from './services/ads.service';

@Component({ /* ... */ })
export class AppComponent implements OnInit {
  private ads = inject(AdsService);

  async ngOnInit() {
    await gatherConsent();
    await this.ads.initialize();
  }
}
```

```typescript
// settings/settings.page.ts — add a row
import { showPrivacyOptions } from '../utils/consent';

async openPrivacyOptions() {
  await showPrivacyOptions();
}
```

## React

```tsx
// App.tsx
import { useEffect } from 'react';
import { gatherConsent } from './utils/consent';
import { initializeAdMob } from './utils/admob';

const App: React.FC = () => {
  useEffect(() => {
    (async () => {
      await gatherConsent();
      await initializeAdMob();
    })();
  }, []);
  // ...
};
```

```tsx
// pages/SettingsPage.tsx — add an item
import { showPrivacyOptions } from '../utils/consent';

<IonItem button onClick={showPrivacyOptions}>
  <IonIcon icon={shield} slot="start" />
  <IonLabel>{t('settings.privacy')}</IonLabel>
</IonItem>
```

## Vue

```vue
<!-- App.vue -->
<script setup lang="ts">
import { onMounted } from 'vue';
import { gatherConsent } from './utils/consent';
import { initializeAdMob } from './utils/admob';

onMounted(async () => {
  await gatherConsent();
  await initializeAdMob();
});
</script>
```

```vue
<!-- views/SettingsPage.vue — add an item -->
<script setup lang="ts">
import { showPrivacyOptions } from '../utils/consent';
</script>

<template>
  <ion-item button @click="showPrivacyOptions">
    <ion-icon name="shield" slot="start" />
    <ion-label>{{ t('settings.privacy') }}</ion-label>
  </ion-item>
</template>
```

## Translation keys to add

Append to `en.json` / `tr.json` (see [`../../ionic-shared/references/localization-content.md`](../../ionic-shared/references/localization-content.md)):

```json
{
  "settings": {
    "privacy": "Privacy options"   // Gizlilik seçenekleri
  }
}
```
