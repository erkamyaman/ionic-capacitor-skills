# Framework Integration Snippets

ATT must fire **after onboarding** and **before** AdMob initialization. The cleanest place is at the moment onboarding completes — where the user navigates from Onboarding → Paywall (or Onboarding → Tabs).

## Angular

```typescript
// onboarding/onboarding.page.ts
import { requestAttPermission } from '../utils/att';
import { gatherConsent } from '../utils/consent';
import { AdsService } from '../services/ads.service';

async completeOnboarding() {
  await this.onboardingService.setCompleted(true);
  await requestAttPermission();   // 1. ATT first
  await gatherConsent();          // 2. UMP next
  await this.adsService.initialize(); // 3. AdMob last
  this.router.navigateByUrl('/paywall', { replaceUrl: true });
}
```

## React

```tsx
// pages/OnboardingPage.tsx
import { requestAttPermission } from '../utils/att';
import { gatherConsent } from '../utils/consent';
import { initializeAdMob } from '../utils/admob';

const completeOnboarding = async () => {
  await setCompleted(true);
  await requestAttPermission();
  await gatherConsent();
  await initializeAdMob();
  router.push('/paywall', 'forward', 'replace');
};
```

## Vue

```vue
<script setup lang="ts">
import { requestAttPermission } from '../utils/att';
import { gatherConsent } from '../utils/consent';
import { initializeAdMob } from '../utils/admob';

async function completeOnboarding() {
  await setCompleted(true);
  await requestAttPermission();
  await gatherConsent();
  await initializeAdMob();
  router.replace('/paywall');
}
</script>
```

## Don't do this

```typescript
// ❌ Apple rejects this
async ngOnInit() {
  await requestAttPermission(); // Way too early — user hasn't seen the app yet
}
```

Move the call out of `App.tsx` / `AppComponent.ngOnInit` / `App.vue` `onMounted` and into the onboarding-completion handler.
