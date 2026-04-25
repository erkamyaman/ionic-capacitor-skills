# Routing (Vue)

`@ionic/vue-router` (built on top of `vue-router`). The onboarding guard is a `router.beforeEach` hook.

## `router/index.ts`

```typescript
import { createRouter, createWebHistory } from '@ionic/vue-router';
import { RouteRecordRaw } from 'vue-router';
import { isOnboardingCompleted } from '../utils/onboarding';
import TabsLayout from '../views/TabsLayout.vue';

const routes: RouteRecordRaw[] = [
  {
    path: '/onboarding',
    component: () => import('../views/OnboardingPage.vue'),
  },
  {
    path: '/paywall',
    component: () => import('../views/PaywallPage.vue'),
  },
  {
    path: '/tabs',
    component: TabsLayout,
    children: [
      { path: '', redirect: '/tabs/home' },
      { path: 'home', component: () => import('../views/HomePage.vue') },
      { path: 'explore', component: () => import('../views/ExplorePage.vue') },
      { path: 'settings', component: () => import('../views/SettingsPage.vue') },
    ],
  },
  { path: '/', redirect: '/tabs' },
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});

router.beforeEach(async (to, _from, next) => {
  if (to.path.startsWith('/tabs') || to.path === '/') {
    const completed = await isOnboardingCompleted();
    if (!completed) {
      return next('/onboarding');
    }
  }
  next();
});

export default router;
```

## Conventions

- All views except `TabsLayout` are dynamic-imported for code-splitting.
- The onboarding guard is a single `beforeEach` hook (Vue's idiomatic pattern). See [onboarding-guard.md](onboarding-guard.md).
- The default route `'/'` redirects to `/tabs`; the guard then redirects to `/onboarding` if needed.

## Navigation API

```typescript
import { useRouter } from 'vue-router';

const router = useRouter();
router.replace('/paywall');
router.push('/settings');
```

Use `replace()` for guard redirects (equivalent to Angular's `replaceUrl: true` and React's `'replace'` action) so the back button doesn't return to the page being skipped.
