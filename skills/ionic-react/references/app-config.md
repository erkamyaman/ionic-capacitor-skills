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
import { useEffect } from 'react';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';
import OnboardingPage from './pages/OnboardingPage';
import PaywallPage from './pages/PaywallPage';
import TabsLayout from './components/TabsLayout';
import { OnboardingGuard } from './components/OnboardingGuard';
import { initializeAdMob } from './utils/admob';

setupIonicReact({ mode: 'md' });

const App: React.FC = () => {
  useEffect(() => {
    initializeAdMob();
  }, []);

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
