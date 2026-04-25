# App Configuration (React)

## `main.tsx`

```tsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

/* Core CSS required for Ionic components */
import '@ionic/react/css/core.css';
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';
import '@ionic/react/css/padding.css';
import '@ionic/react/css/float-elements.css';
import '@ionic/react/css/text-alignment.css';
import '@ionic/react/css/text-transformation.css';
import '@ionic/react/css/flex-utils.css';
import '@ionic/react/css/display.css';
import '@ionic/react/css/palettes/dark.class.css';

import './theme/variables.css';
import './i18n';

const root = createRoot(document.getElementById('root')!);
root.render(<App />);
```

## `App.tsx`

```tsx
import { useEffect, useState } from 'react';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';
import OnboardingPage from './pages/OnboardingPage';
import PaywallPage from './pages/PaywallPage';
import TabsLayout from './components/TabsLayout';
import { OnboardingGuard } from './components/OnboardingGuard';
import { initializeAdMob } from './utils/admob';
import { initializePurchases } from './utils/purchases';
import { applyTheme, getTheme } from './utils/theme';

// `mode: 'md'` forces Material Design styling on iOS too — gives a consistent
// look across both platforms. Drop the option to let Ionic auto-detect (iOS
// users will see iOS-native components, Android users see Material). Pick the
// behavior that fits your design — the rest of this scaffold works either way.
setupIonicReact({ mode: 'md' });

const App: React.FC = () => {
  const [ready, setReady] = useState(false);

  useEffect(() => {
    // Initialize the four shared concerns in parallel.
    // initializePurchases() MUST complete before isPremiumUser() is called
    // (e.g., from TabsLayout's banner-show effect), otherwise the banner
    // shows briefly to premium users.
    Promise.all([
      getTheme().then(applyTheme),
      initializePurchases(),
      initializeAdMob(),
    ]).finally(() => setReady(true));
  }, []);

  // Don't render routes until init completes — prevents flash of incorrect
  // banner state and ensures the onboarding guard's storage read succeeds.
  if (!ready) return null;

  return (
    <IonApp>
      <IonReactRouter>
        <IonRouterOutlet>
          <Route exact path="/onboarding" component={OnboardingPage} />
          <Route exact path="/paywall" component={PaywallPage} />
          <Route path="/tabs">
            <OnboardingGuard>
              <TabsLayout />
            </OnboardingGuard>
          </Route>
          <Route exact path="/">
            <Redirect to="/tabs" />
          </Route>
        </IonRouterOutlet>
      </IonReactRouter>
    </IonApp>
  );
};

export default App;
```

The shared utils each `init*` call uses live in `../../ionic-shared/references/`:

- `getTheme` / `applyTheme` → [theming.md](../../ionic-shared/references/theming.md)
- `initializePurchases` → [revenuecat.md](../../ionic-shared/references/revenuecat.md)
- `initializeAdMob` → [admob.md](../../ionic-shared/references/admob.md)

Onboarding state (`isOnboardingCompleted`) is read on-demand by the `OnboardingGuard` component — no separate `initialize()` call needed for it.

## `i18n/index.ts`

```typescript
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import en from './en.json';
import tr from './tr.json';

const browserLang = navigator.language.split('-')[0];

i18n.use(initReactI18next).init({
  resources: {
    en: { translation: en },
    tr: { translation: tr },
  },
  lng: ['en', 'tr'].includes(browserLang) ? browserLang : 'en',
  fallbackLng: 'en',
  interpolation: { escapeValue: false },
});

export default i18n;
```

`initializeAdMob()` is the framework-agnostic util from `ionic-shared` — see [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md).
