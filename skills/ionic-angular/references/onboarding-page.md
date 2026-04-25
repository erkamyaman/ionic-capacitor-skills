# Onboarding Page (Angular)

Fullscreen page with an HTML5 `<video>` background, gradient overlay, and Swiper slides on top. On completion, the user is sent to the paywall.

## Hard rules

- Use a real HTML5 `<video>` element. **Not** a canvas, **not** a GIF, **not** an embedded player.
- Three separate files: `onboarding.page.html`, `onboarding.page.ts`, `onboarding.page.scss`.

## `onboarding/onboarding.page.html`

```html
<ion-content [fullscreen]="true" class="onboarding-content">
  <video
    [src]="videoUrl"
    autoplay
    loop
    muted
    playsinline
    class="background-video"
  ></video>
  <div class="gradient-overlay"></div>
  <div class="onboarding-slides">
    <!-- Swiper slides go here -->
  </div>
</ion-content>
```

## `onboarding/onboarding.page.scss`

```scss
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
```

## `onboarding/onboarding.page.ts`

```typescript
import { Component } from '@angular/core';
import { IonContent, IonButton, IonIcon } from '@ionic/angular/standalone';
import { Router } from '@angular/router';
import { OnboardingService } from '../services/onboarding.service';

const VIDEO_URL =
  'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4';

@Component({
  selector: 'app-onboarding',
  standalone: true,
  imports: [IonContent, IonButton, IonIcon],
  templateUrl: './onboarding.page.html',
  styleUrls: ['./onboarding.page.scss'],
})
export class OnboardingPage {
  videoUrl = VIDEO_URL;

  constructor(
    private router: Router,
    private onboardingService: OnboardingService,
  ) {}

  async completeOnboarding() {
    await this.onboardingService.setCompleted(true);
    this.router.navigateByUrl('/paywall', { replaceUrl: true });
  }
}
```

Replace `VIDEO_URL` with a hosted asset (or `assets/` file) appropriate to the app. Big Buck Bunny is just a placeholder.

## Flow

```
Onboarding → (setCompleted true + navigate replace) → Paywall → Tabs
```

`replaceUrl: true` matters — without it the back button takes the user back to onboarding.
