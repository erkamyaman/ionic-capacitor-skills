# Persistent Storage (`@capacitor/preferences`)

The single approved way to persist key-value data across web and native. Do **not** use `localStorage`, `@ionic/storage`, or `IndexedDB` for app preferences.

## API

```typescript
import { Preferences } from '@capacitor/preferences';

// Set
await Preferences.set({ key: 'onboardingCompleted', value: 'true' });

// Get (always returns a string or null)
const { value } = await Preferences.get({ key: 'onboardingCompleted' });

// Remove
await Preferences.remove({ key: 'onboardingCompleted' });

// Clear everything (rarely used — be careful)
await Preferences.clear();
```

Values are always strings. Coerce to / from `string` at the boundary — store booleans as `'true'`/`'false'`, numbers via `String()` / `Number()`, objects via `JSON.stringify` / `JSON.parse`.

## Onboarding state utility

```typescript
// utils/onboarding.ts
import { Preferences } from '@capacitor/preferences';

const KEY = 'onboardingCompleted';

export async function isOnboardingCompleted(): Promise<boolean> {
  const { value } = await Preferences.get({ key: KEY });
  return value === 'true';
}

export async function setOnboardingCompleted(completed: boolean): Promise<void> {
  await Preferences.set({ key: KEY, value: String(completed) });
}

export async function resetOnboarding(): Promise<void> {
  await Preferences.remove({ key: KEY });
}
```

This file is **framework-agnostic** — Angular wraps it in a service, React in a hook, Vue in a composable.
