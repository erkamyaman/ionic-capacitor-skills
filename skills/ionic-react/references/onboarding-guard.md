# Onboarding Guard (React)

Component wrapper. Checks the onboarding flag on mount; renders nothing until the check completes; redirects to `/onboarding` if incomplete.

## `components/OnboardingGuard.tsx`

```tsx
import { useEffect, useState } from 'react';
import { useIonRouter } from '@ionic/react';
import { isOnboardingCompleted } from '../utils/onboarding';

export const OnboardingGuard: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const router = useIonRouter();
  const [checked, setChecked] = useState(false);

  useEffect(() => {
    isOnboardingCompleted().then((completed) => {
      if (!completed) {
        router.push('/onboarding', 'forward', 'replace');
      } else {
        setChecked(true);
      }
    });
  }, [router]);

  return checked ? <>{children}</> : null;
};
```

The `[router]` dependency keeps `react-hooks/exhaustive-deps` quiet. `router` is stable across renders so this never re-fires.

## Usage

In `App.tsx`, wrap any protected subtree:

```tsx
<Route path="/tabs">
  <OnboardingGuard>
    <TabsLayout />
  </OnboardingGuard>
</Route>
```

## Why a component, not a hook

`react-router` doesn't have a route-guard primitive like Angular's `CanActivateFn`. The conventional pattern is to wrap protected children in a guard component that decides whether to render them or redirect.
