# Paywall Page (Vue)

Shown immediately after onboarding. Two RevenueCat packages (weekly default, yearly with badge) plus a Restore Purchases button.

## `views/PaywallPage.vue`

```vue
<template>
  <ion-page>
    <ion-content :fullscreen="true">
      <div class="paywall-container">
        <h1>{{ t('paywall.title') }}</h1>

        <div class="subscription-options">
          <div
            v-for="option in subscriptionOptions"
            :key="option.id"
            class="option-card"
            :class="{ selected: selectedPlan === option.id }"
            @click="selectedPlan = option.id"
          >
            <ion-badge v-if="option.badge" color="danger">{{ option.badge }}</ion-badge>
            <h3>{{ t(option.title) }}</h3>
            <p>{{ option.price }}</p>
          </div>
        </div>

        <ion-button expand="block" @click="subscribe">
          {{ t('paywall.subscribe') }}
        </ion-button>

        <ion-button fill="clear" @click="skip">
          {{ t('paywall.skip') }}
        </ion-button>

        <ion-button fill="clear" size="small" @click="handleRestore">
          {{ t('paywall.restore') }}
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonPage, IonContent, IonButton, IonBadge } from '@ionic/vue';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';
import { usePurchases } from '../composables/usePurchases';

const { t } = useI18n();
const router = useRouter();
const { restore } = usePurchases();
const selectedPlan = ref('weekly');

// Note: prices below are placeholders — production paywalls should render
// from `pkg.product.priceString` so they're localized + currency-correct.
const subscriptionOptions = [
  { id: 'weekly', title: 'paywall.weekly', price: '$4.99/week', badge: null },
  { id: 'yearly', title: 'paywall.yearly', price: '$129.99/year', badge: '50% OFF' },
];

async function subscribe() {
  // TODO: map selectedPlan → PurchasesPackage from getOfferings()
  //       and call usePurchases().purchase(pkg)
  router.replace('/tabs');
}

function skip() {
  router.replace('/tabs');
}

async function handleRestore() {
  const restored = await restore();
  if (restored) {
    router.replace('/tabs');
  }
}
</script>
```

For purchase wiring (mapping `selectedPlan` to a `PurchasesPackage` via `getOfferings()`), see [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md).
