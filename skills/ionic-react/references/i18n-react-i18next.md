# i18n with `react-i18next`

Configuration lives in `src/i18n/index.ts` and is imported once in `main.tsx` — see [app-config.md](app-config.md). Translation content lives in [`../../ionic-shared/references/localization-content.md`](../../ionic-shared/references/localization-content.md).

## Usage in components

```tsx
import { useTranslation } from 'react-i18next';

const SettingsPage: React.FC = () => {
  const { t } = useTranslation();
  return <h1>{t('settings.title')}</h1>;
};
```

For attribute placeholders:

```tsx
<IonInput placeholder={t('paywall.title')} />
```

## Changing the language at runtime

```tsx
const { i18n } = useTranslation();
i18n.changeLanguage('tr');
```

Persist the selection to `@capacitor/preferences` so it survives restarts:

```typescript
import { Preferences } from '@capacitor/preferences';
await Preferences.set({ key: 'language', value: 'tr' });
```

On startup, the `i18n/index.ts` already detects the browser language. To honor a saved preference, add a check:

```typescript
const { value: saved } = await Preferences.get({ key: 'language' });
if (saved && ['en', 'tr'].includes(saved)) {
  i18n.changeLanguage(saved);
}
```
