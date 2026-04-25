# Onboarding Page (React)

Fullscreen page wrapped in `<IonPage>`, with HTML5 `<video>` background, gradient overlay, and Swiper slides on top.

## Hard rules

- Use a real HTML5 `<video>` element. **Not** a canvas, **not** a GIF, **not** an embedded player.
- Wrap the page in `<IonPage>` for transitions / lifecycle.

## `pages/OnboardingPage.tsx`

```tsx
import { IonContent, IonPage, IonButton, useIonRouter } from '@ionic/react';
import { useOnboarding } from '../hooks/useOnboarding';
import './OnboardingPage.css';

const VIDEO_URL =
  'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4';

const OnboardingPage: React.FC = () => {
  const router = useIonRouter();
  const { setCompleted } = useOnboarding();

  const completeOnboarding = async () => {
    await setCompleted(true);
    router.push('/paywall', 'forward', 'replace');
  };

  return (
    <IonPage>
      <IonContent fullscreen className="onboarding-content">
        <video
          src={VIDEO_URL}
          autoPlay
          loop
          muted
          playsInline
          className="background-video"
        />
        <div className="gradient-overlay" />
        <div className="onboarding-slides">
          {/* Swiper slides go here */}
          <IonButton onClick={completeOnboarding}>Get Started</IonButton>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default OnboardingPage;
```

## `pages/OnboardingPage.css`

```css
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

## Flow

```
Onboarding → setCompleted(true) → push /paywall (replace) → Paywall → Tabs
```

`router.push(path, 'forward', 'replace')` is the equivalent of Angular's `replaceUrl: true` — without it the back button takes the user back to onboarding.
