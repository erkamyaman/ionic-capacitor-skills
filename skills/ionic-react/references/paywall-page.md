# Paywall Page (React)

Shown immediately after onboarding. Two RevenueCat packages (weekly default, yearly with badge) plus a Restore Purchases button.

## `pages/PaywallPage.tsx`

```tsx
import { useState } from 'react';
import {
  IonContent, IonPage, IonButton, IonBadge, useIonRouter,
} from '@ionic/react';
import { useTranslation } from 'react-i18next';
import { usePurchases } from '../hooks/usePurchases';

const subscriptionOptions = [
  { id: 'weekly', title: 'paywall.weekly', price: '$4.99/week', badge: null },
  { id: 'yearly', title: 'paywall.yearly', price: '$129.99/year', badge: '50% OFF' },
];

const PaywallPage: React.FC = () => {
  const { t } = useTranslation();
  const router = useIonRouter();
  const { restore: restorePurchases } = usePurchases();
  const [selectedPlan, setSelectedPlan] = useState('weekly');

  const subscribe = async () => {
    // TODO: map selectedPlan → PurchasesPackage from getOfferings()
    //       and call usePurchases().purchase(pkg)
    router.push('/tabs', 'forward', 'replace');
  };

  const skip = () => {
    router.push('/tabs', 'forward', 'replace');
  };

  const handleRestore = async () => {
    const restored = await restorePurchases();
    if (restored) {
      router.push('/tabs', 'forward', 'replace');
    }
  };

  return (
    <IonPage>
      <IonContent fullscreen>
        <div className="paywall-container">
          <h1>{t('paywall.title')}</h1>

          <div className="subscription-options">
            {subscriptionOptions.map((option) => (
              <div
                key={option.id}
                className={`option-card ${selectedPlan === option.id ? 'selected' : ''}`}
                onClick={() => setSelectedPlan(option.id)}
              >
                {option.badge && <IonBadge color="danger">{option.badge}</IonBadge>}
                <h3>{t(option.title)}</h3>
                <p>{option.price}</p>
              </div>
            ))}
          </div>

          <IonButton expand="block" onClick={subscribe}>
            {t('paywall.subscribe')}
          </IonButton>

          <IonButton fill="clear" onClick={skip}>
            {t('paywall.skip')}
          </IonButton>

          <IonButton fill="clear" size="small" onClick={handleRestore}>
            {t('paywall.restore')}
          </IonButton>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default PaywallPage;
```

For purchase wiring (mapping `selectedPlan` to a `PurchasesPackage` via `getOfferings()`), see [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md).
