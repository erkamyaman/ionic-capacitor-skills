# Onboarding Guard (Angular)

Functional `CanActivateFn`. Redirects to `/onboarding` when the user has not yet completed it, otherwise lets navigation proceed.

```typescript
// guards/onboarding.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { isOnboardingCompleted } from '../utils/onboarding';

export const onboardingGuard: CanActivateFn = async () => {
  const router = inject(Router);

  const completed = await isOnboardingCompleted();

  if (!completed) {
    router.navigateByUrl('/onboarding', { replaceUrl: true });
    return false;
  }

  return true;
};
```

Wire it onto the `tabs` route in `app.routes.ts`:

```typescript
{
  path: 'tabs',
  loadChildren: () => import('./tabs/tabs.routes').then(m => m.tabsRoutes),
  canActivate: [onboardingGuard],
},
```

## Why functional, not class-based

Functional guards (`CanActivateFn`) are the modern Angular idiom — they're simpler, work with `inject()`, and don't require module registration. Don't generate class-based `CanActivate` guards.
