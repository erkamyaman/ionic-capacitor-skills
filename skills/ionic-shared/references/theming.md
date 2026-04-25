# Theme Switching (Light / Dark / System)

Three modes, persisted via `@capacitor/preferences`, applied by toggling the `ion-palette-dark` class on `<html>`.

## Utility (framework-agnostic)

```typescript
// utils/theme.ts
import { Preferences } from '@capacitor/preferences';

export type ThemeMode = 'light' | 'dark' | 'system';

const KEY = 'themeMode';

export async function getTheme(): Promise<ThemeMode> {
  const { value } = await Preferences.get({ key: KEY });
  return (value as ThemeMode) || 'system';
}

export async function setTheme(mode: ThemeMode): Promise<void> {
  await Preferences.set({ key: KEY, value: mode });
  applyTheme(mode);
}

export function applyTheme(mode: ThemeMode): void {
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  const isDark = mode === 'dark' || (mode === 'system' && prefersDark);
  document.documentElement.classList.toggle('ion-palette-dark', isDark);
}
```

## When to call `applyTheme`

- On app startup (so the saved preference is restored).
- Whenever the user changes the theme via the Settings page.
- Optionally on a `prefers-color-scheme` change listener if you want `'system'` mode to react to OS-level changes while the app is open.

## CSS

Ionic ships dark palette CSS that activates when the `ion-palette-dark` class is present on `<html>`. Make sure `@ionic/<framework>/css/palettes/dark.class.css` (or the equivalent) is imported in `main.ts`/`main.tsx`.
