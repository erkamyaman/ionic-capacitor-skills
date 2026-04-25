# Routing (React)

Top-level routes are defined in `App.tsx` (see [app-config.md](app-config.md)). Tab routes are defined inside `TabsLayout.tsx`.

## `components/TabsLayout.tsx`

```tsx
import {
  IonTabs, IonRouterOutlet, IonTabBar, IonTabButton, IonIcon, IonLabel,
} from '@ionic/react';
import { Route, Redirect } from 'react-router-dom';
import { home, compass, settings } from 'ionicons/icons';
import { useTranslation } from 'react-i18next';
import { useEffect } from 'react';
import HomePage from '../pages/HomePage';
import ExplorePage from '../pages/ExplorePage';
import SettingsPage from '../pages/SettingsPage';
import { showBannerAd, hideBannerAd } from '../utils/admob';
import { isPremiumUser } from '../utils/purchases';

const TabsLayout: React.FC = () => {
  const { t } = useTranslation();

  useEffect(() => {
    isPremiumUser().then((premium) => {
      if (!premium) showBannerAd();
    });
    return () => { hideBannerAd(); };
  }, []);

  return (
    <IonTabs>
      <IonRouterOutlet>
        <Route exact path="/tabs/home" component={HomePage} />
        <Route exact path="/tabs/explore" component={ExplorePage} />
        <Route exact path="/tabs/settings" component={SettingsPage} />
        <Route exact path="/tabs">
          <Redirect to="/tabs/home" />
        </Route>
      </IonRouterOutlet>
      <IonTabBar slot="bottom">
        <IonTabButton tab="home" href="/tabs/home">
          <IonIcon icon={home} />
          <IonLabel>{t('tabs.home')}</IonLabel>
        </IonTabButton>
        <IonTabButton tab="explore" href="/tabs/explore">
          <IonIcon icon={compass} />
          <IonLabel>{t('tabs.explore')}</IonLabel>
        </IonTabButton>
        <IonTabButton tab="settings" href="/tabs/settings">
          <IonIcon icon={settings} />
          <IonLabel>{t('tabs.settings')}</IonLabel>
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};

export default TabsLayout;
```

## Conventions

- Each `<Route>` corresponds to a page wrapped in `<IonPage>` so transitions and lifecycle work.
- The `OnboardingGuard` component wraps `<TabsLayout />` in `App.tsx` to redirect when onboarding hasn't been completed — see [onboarding-guard.md](onboarding-guard.md).
- Banner ad lifecycle is managed inside `TabsLayout`'s `useEffect` so the ad shows for non-premium users while in tabs and hides when leaving.

## Why the nested `/tabs/home` routes

`App.tsx` mounts `<Route path="/tabs">` (no `exact`) to give the tabs subtree, then `<TabsLayout>` renders its own `<IonRouterOutlet>` containing the per-tab `<Route exact path="/tabs/home" ...>`. This double-nesting is the standard `@ionic/react-router` shape — the outer outlet handles tab→non-tab transitions, the inner outlet handles tab-to-tab. It looks odd at first but it's what enables the native-feeling stacked navigation.

## Navigation API

```tsx
import { useIonRouter } from '@ionic/react';

const router = useIonRouter();
router.push('/paywall', 'forward', 'replace');
```

`useIonRouter()` integrates with Ionic page transitions. Don't use `react-router-dom`'s `useHistory` directly for in-app navigation — Ionic transitions need the Ionic router hook.
