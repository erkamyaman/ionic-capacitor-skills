# React Best Practices

## Functional components only

❌ Class:

```tsx
class HomePage extends React.Component {
  render() {
    return <IonContent>...</IonContent>;
  }
}
```

✅ Functional + hooks + `<IonPage>`:

```tsx
import {
  IonContent, IonHeader, IonTitle, IonToolbar, IonPage,
} from '@ionic/react';
import { useTranslation } from 'react-i18next';

const HomePage: React.FC = () => {
  const { t } = useTranslation();

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>{t('home.title')}</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
        {/* content */}
      </IonContent>
    </IonPage>
  );
};

export default HomePage;
```

## Always wrap pages in `<IonPage>`

This is the most common React-Ionic mistake. `<IonPage>` is what enables Ionic's page transitions and lifecycle events (`ionViewWillEnter`, etc.). Forget it and you get janky transitions and the page may not render correctly inside a tab outlet.

## Navigation

Use `useIonRouter()` from `@ionic/react`:

```tsx
const router = useIonRouter();
router.push('/paywall', 'forward', 'replace');
```

The arguments are `(path, direction, routerAction)`:

- `direction`: `'forward' | 'back' | 'none'` — controls the transition animation.
- `routerAction`: `'push' | 'replace' | 'pop'` — controls the history stack. Use `'replace'` for guard redirects.

Don't reach for `useHistory` from `react-router-dom` directly — it doesn't trigger Ionic transitions.

## Icons

Import icons from `ionicons/icons` and pass as values, never as strings:

```tsx
import { home } from 'ionicons/icons';

<IonIcon icon={home} />        // ✅
<IonIcon name="home" />        // ❌ (Angular-only pattern)
```

## TypeScript

- `strict: true`, no `any`.
- Type Capacitor plugin responses explicitly.
- For component props, `React.FC<Props>` or explicit prop types — pick one and stay consistent.
