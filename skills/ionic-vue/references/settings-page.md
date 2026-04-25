# Settings Page (Vue)

Required entries:

1. **Language** — switch between supported locales.
2. **Theme** — Light / Dark / System.
3. **Notifications** — toggle that requests permission on enable.
4. **Remove Ads** — navigates to `/paywall`. Hide for premium users.
5. **Reset Onboarding** — clears the flag and routes back to `/onboarding`.

## `views/SettingsPage.vue`

```vue
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>{{ t('settings.title') }}</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        <ion-item>
          <ion-icon :icon="language" slot="start" aria-hidden="true" />
          <ion-label>{{ t('settings.language') }}</ion-label>
          <ion-select :value="currentLang" @ion-change="changeLanguage($event)">
            <ion-select-option value="en">English</ion-select-option>
            <ion-select-option value="tr">Türkçe</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-icon :icon="colorPalette" slot="start" aria-hidden="true" />
          <ion-label>{{ t('settings.theme') }}</ion-label>
          <ion-select :value="currentTheme" @ion-change="changeTheme($event)">
            <ion-select-option value="system">{{ t('settings.system') }}</ion-select-option>
            <ion-select-option value="light">{{ t('settings.light') }}</ion-select-option>
            <ion-select-option value="dark">{{ t('settings.dark') }}</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-icon :icon="notifications" slot="start" aria-hidden="true" />
          <ion-label>{{ t('settings.notifications') }}</ion-label>
          <ion-toggle
            :checked="notificationsEnabled"
            @ion-change="toggleNotifications($event)"
          />
        </ion-item>

        <ion-item v-if="!premium" button @click="removeAds">
          <ion-icon :icon="star" slot="start" aria-hidden="true" />
          <ion-label>{{ t('settings.removeAds') }}</ion-label>
        </ion-item>

        <ion-item button @click="resetOnboardingFlow">
          <ion-icon :icon="refresh" slot="start" aria-hidden="true" />
          <ion-label>{{ t('settings.resetOnboarding') }}</ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonList, IonItem, IonLabel, IonIcon, IonToggle,
  IonSelect, IonSelectOption,
} from '@ionic/vue';
import {
  language, colorPalette, notifications, star, refresh,
} from 'ionicons/icons';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';
import { Preferences } from '@capacitor/preferences';
import { useTheme } from '../composables/useTheme';
import { useOnboarding } from '../composables/useOnboarding';
import { usePurchases } from '../composables/usePurchases';
import { useNotifications } from '../composables/useNotifications';
import type { ThemeMode } from '../utils/theme';

const { t, locale } = useI18n();
const router = useRouter();
// Composables expose reactive refs (signal-store pattern) — no onMounted/ref shadow needed.
const { mode: currentTheme, setMode: setTheme } = useTheme();
const { reset } = useOnboarding();
const { isPremium: premium } = usePurchases();
const { permissionGranted: notificationsEnabled, requestPermission } = useNotifications();

const currentLang = ref<string>(locale.value || 'en');

function changeLanguage(event: CustomEvent) {
  const lang = event.detail.value;
  currentLang.value = lang;
  locale.value = lang;
  Preferences.set({ key: 'language', value: lang });
}

function changeTheme(event: CustomEvent) {
  const mode = event.detail.value as ThemeMode;
  setTheme(mode);
}

async function toggleNotifications(event: CustomEvent) {
  const enabled = event.detail.checked;
  if (enabled) {
    // Request OS permission only when the user opts in. If denied, the
    // composable's permissionGranted ref stays false, reverting the toggle.
    await requestPermission();
  }
  // When toggled off, the OS permission isn't actually revoked (only
  // possible via Settings) — the app just stops calling push APIs.
}

function removeAds() {
  router.push('/paywall');
}

async function resetOnboardingFlow() {
  await reset();
  router.replace('/onboarding');
}
</script>
```
