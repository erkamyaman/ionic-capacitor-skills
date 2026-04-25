# Tabs Navigation (React)

The full `TabsLayout` component (with banner ad lifecycle) is defined in [routing.md](routing.md). This file documents conventions and icon choices.

## Common ionicons

Imported from `ionicons/icons`:

| Purpose       | Import name     |
|---------------|-----------------|
| Home          | `home`          |
| Explore       | `compass`       |
| Settings      | `settings`      |
| Profile       | `person`        |
| Search        | `search`        |
| Favorites     | `heart`         |
| Notifications | `notifications` |

```tsx
import { home, compass, settings } from 'ionicons/icons';

<IonIcon icon={home} />
```

In React, icons are passed as **values** (`icon={home}`), not as **strings** (`name="home"`) — the latter is the Angular pattern.

## Banner ad lifecycle

The banner is shown on tab-layout mount (only for non-premium users) and hidden on unmount. This keeps the ad visible only while the main app is showing — onboarding and paywall pages don't have it. See the `useEffect` block in [routing.md](routing.md).

## Tab order

Stick to the standard order: **Home → Explore → Settings**. Settings stays on the right because users expect it there.
