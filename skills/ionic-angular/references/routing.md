# Routing (Angular)

Lazy-loaded routes with the onboarding guard protecting the tabs section.

## `app.routes.ts`

```typescript
import { Routes } from '@angular/router';
import { onboardingGuard } from './guards/onboarding.guard';

export const routes: Routes = [
  {
    path: 'onboarding',
    loadComponent: () => import('./onboarding/onboarding.page').then(m => m.OnboardingPage),
  },
  {
    path: 'paywall',
    loadComponent: () => import('./paywall/paywall.page').then(m => m.PaywallPage),
  },
  {
    path: 'tabs',
    loadChildren: () => import('./tabs/tabs.routes').then(m => m.tabsRoutes),
    canMatch: [onboardingGuard],   // CanMatchFn — see onboarding-guard.md for why over canActivate
  },
  {
    path: '',
    redirectTo: 'tabs',
    pathMatch: 'full',
  },
];
```

## `tabs/tabs.routes.ts`

```typescript
import { Routes } from '@angular/router';
import { TabsPage } from './tabs.page';

export const tabsRoutes: Routes = [
  {
    path: '',
    component: TabsPage,
    children: [
      {
        path: 'home',
        loadComponent: () => import('../home/home.page').then(m => m.HomePage),
      },
      {
        path: 'explore',
        loadComponent: () => import('../explore/explore.page').then(m => m.ExplorePage),
      },
      {
        path: 'settings',
        loadComponent: () => import('../settings/settings.page').then(m => m.SettingsPage),
      },
      {
        path: '',
        redirectTo: 'home',
        pathMatch: 'full',
      },
    ],
  },
];
```

## Conventions

- All pages lazy via `loadComponent` (or `loadChildren` for nested route trees).
- The onboarding guard is attached to `tabs` and any other authenticated route — it redirects to `/onboarding` when state is missing.
- Default route (`''`) redirects to `tabs`; the guard handles the redirect to `onboarding` when needed.
