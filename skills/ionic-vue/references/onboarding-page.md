# Onboarding Page (Vue)

Fullscreen page wrapped in `<ion-page>`, with HTML5 `<video>` background, gradient overlay, and Swiper slides on top.

## Hard rules

- Use a real HTML5 `<video>` element. **Not** a canvas, **not** a GIF, **not** an embedded player.
- Wrap the page in `<ion-page>` for transitions / lifecycle.

## `views/OnboardingPage.vue`

```vue
<template>
  <ion-page>
    <ion-content :fullscreen="true" class="onboarding-content">
      <video
        :src="videoUrl"
        autoplay
        loop
        muted
        playsinline
        class="background-video"
      />
      <div class="gradient-overlay" />
      <div class="onboarding-slides">
        <!-- Swiper slides go here -->
        <ion-button @click="completeOnboarding">Get Started</ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonContent, IonButton } from '@ionic/vue';
import { useRouter } from 'vue-router';
import { useOnboarding } from '../composables/useOnboarding';

const videoUrl =
  'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4';

const router = useRouter();
const { setCompleted } = useOnboarding();

async function completeOnboarding() {
  await setCompleted(true);
  router.replace('/paywall');
}
</script>

<style scoped>
.background-video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 0;
}

.gradient-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.7));
  z-index: 1;
}

.onboarding-slides {
  position: relative;
  z-index: 2;
  height: 100%;
}
</style>
```

Replace `VIDEO_URL` with a hosted asset (or a file in `src/assets/`) appropriate to the app. Big Buck Bunny is a placeholder.

## Flow

```
Onboarding → setCompleted(true) → router.replace('/paywall') → Paywall → Tabs
```

`router.replace()` (instead of `push`) ensures the back button doesn't return to onboarding.
