# Settings Page (React)

Required entries:

1. **Language** — switch between supported locales.
2. **Theme** — Light / Dark / System.
3. **Notifications** — toggle that requests permission on enable.
4. **Remove Ads** — navigates to `/paywall`. Hide for premium users.
5. **Reset Onboarding** — clears the flag and routes back to `/onboarding`.

## `pages/SettingsPage.tsx`

```tsx
import { useState, useEffect } from 'react';
import {
  IonContent, IonHeader, IonTitle, IonToolbar, IonPage,
  IonList, IonItem, IonLabel, IonIcon, IonToggle,
  IonSelect, IonSelectOption, useIonRouter,
} from '@ionic/react';
import {
  language, colorPalette, notifications, star, refresh,
} from 'ionicons/icons';
import { useTranslation } from 'react-i18next';
import { Preferences } from '@capacitor/preferences';
import { useTheme } from '../hooks/useTheme';
import { useOnboarding } from '../hooks/useOnboarding';
import { usePurchases } from '../hooks/usePurchases';
import { ThemeMode } from '../utils/theme';

const SettingsPage: React.FC = () => {
  const { t, i18n } = useTranslation();
  const router = useIonRouter();
  const { getTheme, setTheme } = useTheme();
  const { reset } = useOnboarding();
  const { isPremium } = usePurchases();

  const [currentLang, setCurrentLang] = useState('en');
  const [currentTheme, setCurrentTheme] = useState<ThemeMode>('system');
  const [notificationsEnabled, setNotificationsEnabled] = useState(false);
  const [premium, setPremium] = useState(false);

  useEffect(() => {
    setCurrentLang(i18n.language || 'en');
    getTheme().then(setCurrentTheme);
    isPremium().then(setPremium);
  }, []);

  const changeLanguage = (lang: string) => {
    setCurrentLang(lang);
    i18n.changeLanguage(lang);
    Preferences.set({ key: 'language', value: lang });
  };

  const changeTheme = (mode: ThemeMode) => {
    setCurrentTheme(mode);
    setTheme(mode);
  };

  const resetOnboarding = async () => {
    await reset();
    router.push('/onboarding', 'forward', 'replace');
  };

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>{t('settings.title')}</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
        <IonList>
          <IonItem>
            <IonIcon icon={language} slot="start" />
            <IonLabel>{t('settings.language')}</IonLabel>
            <IonSelect
              value={currentLang}
              onIonChange={(e) => changeLanguage(e.detail.value)}
            >
              <IonSelectOption value="en">English</IonSelectOption>
              <IonSelectOption value="tr">Türkçe</IonSelectOption>
            </IonSelect>
          </IonItem>

          <IonItem>
            <IonIcon icon={colorPalette} slot="start" />
            <IonLabel>{t('settings.theme')}</IonLabel>
            <IonSelect
              value={currentTheme}
              onIonChange={(e) => changeTheme(e.detail.value)}
            >
              <IonSelectOption value="system">{t('settings.system')}</IonSelectOption>
              <IonSelectOption value="light">{t('settings.light')}</IonSelectOption>
              <IonSelectOption value="dark">{t('settings.dark')}</IonSelectOption>
            </IonSelect>
          </IonItem>

          <IonItem>
            <IonIcon icon={notifications} slot="start" />
            <IonLabel>{t('settings.notifications')}</IonLabel>
            <IonToggle
              checked={notificationsEnabled}
              onIonChange={(e) => setNotificationsEnabled(e.detail.checked)}
            />
          </IonItem>

          {!premium && (
            <IonItem button onClick={() => router.push('/paywall')}>
              <IonIcon icon={star} slot="start" />
              <IonLabel>{t('settings.removeAds')}</IonLabel>
            </IonItem>
          )}

          <IonItem button onClick={resetOnboarding}>
            <IonIcon icon={refresh} slot="start" />
            <IonLabel>{t('settings.resetOnboarding')}</IonLabel>
          </IonItem>
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default SettingsPage;
```
