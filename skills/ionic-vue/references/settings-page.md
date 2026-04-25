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
          <ion-icon name="language" slot="start" />
          <ion-label>{{ t('settings.language') }}</ion-label>
          <ion-select :value="currentLang" @ion-change="changeLanguage($event)">
            <ion-select-option value="en">English</ion-select-option>
            <ion-select-option value="tr">Türkçe</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-icon name="color-palette" slot="start" />
          <ion-label>{{ t('settings.theme') }}</ion-label>
          <ion-select :value="currentTheme" @ion-change="changeTheme($event)">
            <ion-select-option value="system">{{ t('settings.system') }}</ion-select-option>
            <ion-select-option value="light">{{ t('settings.light') }}</ion-select-option>
            <ion-select-option value="dark">{{ t('settings.dark') }}</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-icon name="notifications" slot="start" />
          <ion-label>{{ t('settings.notifications') }}</ion-label>
          <ion-toggle
            :checked="notificationsEnabled"
            @ion-change="toggleNotifications($event)"
          />
        </ion-item>

        <ion-item v-if="!premium" button @click="removeAds">
          <ion-icon name="star" slot="start" />
          <ion-label>{{ t('settings.removeAds') }}</ion-label>
        </ion-item>

        <ion-item button @click="resetOnboardingFlow">
          <ion-icon name="refresh" slot="start" />
          <ion-label>{{ t('settings.resetOnboarding') }}</ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonList, IonItem, IonLabel, IonIcon, IonToggle,
  IonSelect, IonSelectOption,
} from '@ionic/vue';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';
import { Preferences } from '@capacitor/preferences';
import { useTheme } from '../composables/useTheme';
import { useOnboarding } from '../composables/useOnboarding';
import { usePurchases } from '../composables/usePurchases';
import type { ThemeMode } from '../utils/theme';

const { t, locale } = useI18n();
const router = useRouter();
const { getTheme, setTheme } = useTheme();
const { reset } = useOnboarding();
const { isPremium } = usePurchases();

const currentLang = ref('en');
const currentTheme = ref<ThemeMode>('system');
const notificationsEnabled = ref(false);
const premium = ref(false);

onMounted(async () => {
  currentLang.value = locale.value || 'en';
  currentTheme.value = await getTheme();
  premium.value = await isPremium();
});

function changeLanguage(event: CustomEvent) {
  const lang = event.detail.value;
  currentLang.value = lang;
  locale.value = lang;
  Preferences.set({ key: 'language', value: lang });
}

function changeTheme(event: CustomEvent) {
  const mode = event.detail.value as ThemeMode;
  currentTheme.value = mode;
  setTheme(mode);
}

function toggleNotifications(event: CustomEvent) {
  notificationsEnabled.value = event.detail.checked;
  // call useNotifications().requestPermission() here on enable
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
