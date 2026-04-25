# Settings Page (Angular)

Required entries:

1. **Language** — switch between supported locales.
2. **Theme** — Light / Dark / System.
3. **Notifications** — toggle that requests permission on enable.
4. **Remove Ads** — navigates to `/paywall`. Hide for premium users.
5. **Reset Onboarding** — clears the flag and routes back to `/onboarding`. Useful for QA / demo.

## `settings/settings.page.html`

```html
<ion-header>
  <ion-toolbar>
    <ion-title>{{ 'settings.title' | translate }}</ion-title>
  </ion-toolbar>
</ion-header>
<ion-content>
  <ion-list>
    <ion-item>
      <ion-icon name="language" slot="start"></ion-icon>
      <ion-label>{{ 'settings.language' | translate }}</ion-label>
      <ion-select [value]="currentLang" (ionChange)="changeLanguage($event)">
        <ion-select-option value="en">English</ion-select-option>
        <ion-select-option value="tr">Türkçe</ion-select-option>
      </ion-select>
    </ion-item>

    <ion-item>
      <ion-icon name="color-palette" slot="start"></ion-icon>
      <ion-label>{{ 'settings.theme' | translate }}</ion-label>
      <ion-select [value]="currentTheme" (ionChange)="changeTheme($event)">
        <ion-select-option value="system">{{ 'settings.system' | translate }}</ion-select-option>
        <ion-select-option value="light">{{ 'settings.light' | translate }}</ion-select-option>
        <ion-select-option value="dark">{{ 'settings.dark' | translate }}</ion-select-option>
      </ion-select>
    </ion-item>

    <ion-item>
      <ion-icon name="notifications" slot="start"></ion-icon>
      <ion-label>{{ 'settings.notifications' | translate }}</ion-label>
      <ion-toggle
        [checked]="notificationsEnabled"
        (ionChange)="toggleNotifications($event)"
      ></ion-toggle>
    </ion-item>

    <ion-item *ngIf="!isPremium" button (click)="removeAds()">
      <ion-icon name="star" slot="start"></ion-icon>
      <ion-label>{{ 'settings.removeAds' | translate }}</ion-label>
    </ion-item>

    <ion-item button (click)="resetOnboarding()">
      <ion-icon name="refresh" slot="start"></ion-icon>
      <ion-label>{{ 'settings.resetOnboarding' | translate }}</ion-label>
    </ion-item>
  </ion-list>
</ion-content>
```

## `settings/settings.page.ts`

```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import {
  IonContent, IonHeader, IonTitle, IonToolbar,
  IonList, IonItem, IonLabel, IonIcon, IonToggle,
  IonSelect, IonSelectOption,
} from '@ionic/angular/standalone';
import { NgIf } from '@angular/common';
import { TranslateModule, TranslateService } from '@ngx-translate/core';
import { Preferences } from '@capacitor/preferences';
import { ThemeService } from '../services/theme.service';
import { OnboardingService } from '../services/onboarding.service';
import { PurchasesService } from '../services/purchases.service';

@Component({
  selector: 'app-settings',
  standalone: true,
  imports: [
    IonContent, IonHeader, IonTitle, IonToolbar,
    IonList, IonItem, IonLabel, IonIcon, IonToggle,
    IonSelect, IonSelectOption,
    NgIf, TranslateModule,
  ],
  templateUrl: './settings.page.html',
  styleUrls: ['./settings.page.scss'],
})
export class SettingsPage implements OnInit {
  currentLang = 'en';
  currentTheme = 'system';
  notificationsEnabled = false;
  isPremium = false;

  constructor(
    private router: Router,
    private themeService: ThemeService,
    private onboardingService: OnboardingService,
    private purchasesService: PurchasesService,
    private translate: TranslateService,
  ) {}

  async ngOnInit() {
    this.currentLang = this.translate.currentLang || 'en';
    this.currentTheme = await this.themeService.getTheme();
    this.isPremium = await this.purchasesService.isPremium();
  }

  changeLanguage(event: CustomEvent) {
    const lang = event.detail.value;
    this.translate.use(lang);
    Preferences.set({ key: 'language', value: lang });
  }

  changeTheme(event: CustomEvent) {
    this.themeService.setTheme(event.detail.value);
  }

  toggleNotifications(event: CustomEvent) {
    this.notificationsEnabled = event.detail.checked;
    // call NotificationsService.requestPermission() here on enable
  }

  removeAds() {
    this.router.navigateByUrl('/paywall');
  }

  async resetOnboarding() {
    await this.onboardingService.reset();
    this.router.navigateByUrl('/onboarding', { replaceUrl: true });
  }
}
```
