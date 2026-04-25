# Tabs Navigation (Vue)

`ion-tabs` + `ion-tab-bar`. Three tabs by default: Home, Explore, Settings. Show banner on enter; hide on leave.

## Common ionicons

Imported from `ionicons/icons`:

| Purpose       | Import name     |
|---------------|-----------------|
| Home          | `home`          |
| Explore       | `compass`       |
| Settings      | `settings`      |
| Profile       | `person`        |
| Search        | `search`        |
| Favorites     | `heart`         |
| Notifications | `notifications` |

```typescript
import { home, compass, settings } from 'ionicons/icons';
```

## `views/TabsLayout.vue`

```vue
<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet />
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="home" href="/tabs/home">
          <ion-icon :icon="home" />
          <ion-label>{{ t('tabs.home') }}</ion-label>
        </ion-tab-button>
        <ion-tab-button tab="explore" href="/tabs/explore">
          <ion-icon :icon="compass" />
          <ion-label>{{ t('tabs.explore') }}</ion-label>
        </ion-tab-button>
        <ion-tab-button tab="settings" href="/tabs/settings">
          <ion-icon :icon="settings" />
          <ion-label>{{ t('tabs.settings') }}</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonTabs, IonRouterOutlet, IonTabBar, IonTabButton, IonIcon, IonLabel,
} from '@ionic/vue';
import { home, compass, settings } from 'ionicons/icons';
import { useI18n } from 'vue-i18n';
import { onMounted, onUnmounted } from 'vue';
import { showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

const { t } = useI18n();

onMounted(async () => {
  const premium = await isPremiumUser();
  if (!premium) await showBannerAd();
});

onUnmounted(async () => {
  await hideBannerAd();
});
</script>
```

## Icons in Vue

Pass icons as **bound values** via `:icon="home"`, the same as React. Don't use `name="home"` — that's the Angular pattern.

## Banner ad lifecycle

Shown on tabs mount (only for non-premium users), hidden on unmount. This keeps the ad visible only inside the main app.
