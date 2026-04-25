# Onboarding Guard (Angular)

Functional `CanMatchFn` reading the onboarding flag from `OnboardingService`'s signal store and redirecting to `/onboarding` when it's missing. Uses `CanMatch` (not `CanActivate`) so the lazy `tabs` chunk never loads on a guard rejection — no flash of `/tabs` content while the redirect resolves.

## The guard

```typescript
// guards/onboarding.guard.ts
import { inject } from '@angular/core';
import { CanMatchFn, Router } from '@angular/router';
import { OnboardingService } from '../services/onboarding.service';

export const onboardingGuard: CanMatchFn = () => {
  const router = inject(Router);
  const onboarding = inject(OnboardingService);

  if (!onboarding.completed()) {
    return router.parseUrl('/onboarding');   // returning a UrlTree is the
                                              // idiomatic way to redirect
                                              // from a guard — no imperative
                                              // router.navigateByUrl side effect.
  }
  return true;
};
```

`onboarding.completed()` is a synchronous signal read — fast, no `await`. The signal is hydrated by `OnboardingService.initialize()`, which runs inside an `APP_INITIALIZER` (see [app-config.md](app-config.md)). Because `APP_INITIALIZER` blocks Angular bootstrap until its Promise resolves, **the guard is guaranteed to see the hydrated value** — there's no race.

(Earlier iterations of this scaffold ran initialization in `AppComponent.ngOnInit`, which fires *after* the router has already started matching the initial URL. That window let the guard read a default-`false` signal and redirect returning users back to onboarding. Moving init into `APP_INITIALIZER` closes that window — the guarantee is real, not best-effort.)

## Wire it onto the `tabs` route

```typescript
// app.routes.ts
{
  path: 'tabs',
  loadChildren: () => import('./tabs/tabs.routes').then(m => m.tabsRoutes),
  canMatch: [onboardingGuard],
},
```

`canMatch: [onboardingGuard]` (not `canActivate`) is what gives you the flash-free experience.

## Why functional

Functional guards (`CanMatchFn` / `CanActivateFn`) are the modern Angular idiom — they're concise, work with `inject()`, and don't require module registration. Don't generate class-based `CanMatch` / `CanActivate` guards.

## Why `CanMatch` over `CanActivate`

- `CanActivate` runs **after** the lazy chunk is fetched and route children are matched. A failing guard then triggers the redirect — but the user briefly sees the activated route's outlet during the navigation.
- `CanMatch` runs **before** matching. The lazy chunk doesn't fetch, no route activates, and the redirect happens cleanly.

For navigations gated by route data (params, resolvers), `CanActivate` is still appropriate. For "user must have done X to be in this section," `CanMatch` is the better fit.

## Returning `UrlTree` vs imperative `navigateByUrl`

The guard above returns `router.parseUrl('/onboarding')` rather than calling `router.navigateByUrl(...)` and returning `false`. Both work, but `UrlTree` is the canonical pattern — Angular treats it as "deny + redirect to this URL atomically," eliminating the brief intermediate state where the original navigation is rejected before the redirect navigation starts.

## Alternative: stateless guard reading Preferences directly

If you'd rather not depend on `APP_INITIALIZER` ordering at all (e.g., a feature route owned by a sub-team that doesn't want to touch app config), the guard can read `@capacitor/preferences` directly per navigation. This is the pattern the React and Vue siblings use:

```typescript
import { inject } from '@angular/core';
import { CanMatchFn, Router } from '@angular/router';
import { isOnboardingCompleted } from '../utils/onboarding';

export const onboardingGuard: CanMatchFn = async () => {
  const router = inject(Router);
  const completed = await isOnboardingCompleted();

  return completed ? true : router.parseUrl('/onboarding');
};
```

Costs a Preferences read on every navigation into `/tabs` (cheap, but not free). The signal-store version is faster and is the canonical Angular path; this fallback is here if you ever need to decouple the guard from app-bootstrap discipline.
