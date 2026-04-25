# Onboarding Guard (Vue)

Vue uses a single global `router.beforeEach` hook. The full code is in [routing.md](routing.md); this file describes intent and edge cases.

## The guard

```typescript
router.beforeEach(async (to, _from, next) => {
  if (to.path.startsWith('/tabs') || to.path === '/') {
    const completed = await isOnboardingCompleted();
    if (!completed) {
      return next('/onboarding');
    }
  }
  next();
});
```

## Behavior

- Triggered before **every** navigation.
- Only enforces onboarding for `/tabs/*` routes and the root `/`.
- `/onboarding` and `/paywall` are explicitly **not** guarded — onboarding is reachable, paywall is reachable from settings → "Remove Ads".
- `next('/onboarding')` redirects; `next()` allows the original navigation.

## Why this pattern (vs Angular's per-route `canActivate`)

`vue-router` doesn't have a per-route guard like Angular's `CanActivateFn`. The idiomatic pattern is one global `beforeEach` that decides based on the destination path. This keeps the logic in one place and is easy to test.

## Adding more guards

For additional protected sections, extend the path check:

```typescript
const protectedPaths = ['/tabs', '/'];
if (protectedPaths.some(p => to.path.startsWith(p))) {
  // ...
}
```

Or, for more complex cases, attach `meta: { requiresOnboarding: true }` to specific routes and check `to.matched.some(r => r.meta.requiresOnboarding)`.
