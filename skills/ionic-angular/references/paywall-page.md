# Paywall Page (Angular)

Shown immediately after onboarding. Two RevenueCat subscription packages (weekly default, yearly with 50% OFF badge) plus a Restore Purchases button.

## Required actions

- **Subscribe** — purchase the selected package via RevenueCat.
- **Continue with ads** — skip to `/tabs` without buying.
- **Restore Purchases** — required by Apple. Restores any prior purchase for the signed-in store account.

## `paywall/paywall.page.html`

```html
<ion-content [fullscreen]="true">
  <div class="paywall-container">
    <h1>{{ 'paywall.title' | translate }}</h1>

    <div class="subscription-options">
      <div
        *ngFor="let option of subscriptionOptions"
        class="option-card"
        [class.selected]="selectedPlan === option.id"
        (click)="selectedPlan = option.id"
      >
        <ion-badge *ngIf="option.badge" color="danger">{{ option.badge }}</ion-badge>
        <h3>{{ option.title | translate }}</h3>
        <p>{{ option.price }}</p>
      </div>
    </div>

    <ion-button expand="block" (click)="subscribe()">
      {{ 'paywall.subscribe' | translate }}
    </ion-button>

    <ion-button fill="clear" (click)="skip()">
      {{ 'paywall.skip' | translate }}
    </ion-button>

    <ion-button fill="clear" size="small" (click)="restore()">
      {{ 'paywall.restore' | translate }}
    </ion-button>
  </div>
</ion-content>
```

## `paywall/paywall.page.scss`

```scss
.paywall-container { /* layout / spacing */ }
.subscription-options { /* grid of option cards */ }
.option-card {
  &.selected { /* selected state */ }
}
```

## `paywall/paywall.page.ts`

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';
import {
  IonContent, IonButton, IonIcon, IonBadge,
} from '@ionic/angular/standalone';
import { NgFor, NgIf, NgClass } from '@angular/common';
import { TranslateModule } from '@ngx-translate/core';
import { PurchasesService } from '../services/purchases.service';

@Component({
  selector: 'app-paywall',
  standalone: true,
  imports: [
    IonContent, IonButton, IonIcon, IonBadge,
    NgFor, NgIf, NgClass, TranslateModule,
  ],
  templateUrl: './paywall.page.html',
  styleUrls: ['./paywall.page.scss'],
})
export class PaywallPage {
  selectedPlan = 'weekly';

  subscriptionOptions = [
    { id: 'weekly', title: 'paywall.weekly', price: '$4.99/week', badge: null },
    { id: 'yearly', title: 'paywall.yearly', price: '$129.99/year', badge: '50% OFF' },
  ];

  constructor(
    private router: Router,
    private purchasesService: PurchasesService,
  ) {}

  async subscribe() {
    // TODO: map selectedPlan → PurchasesPackage from getOfferings()
    //       and call this.purchasesService.purchase(pkg)
    this.router.navigateByUrl('/tabs', { replaceUrl: true });
  }

  skip() {
    this.router.navigateByUrl('/tabs', { replaceUrl: true });
  }

  async restore() {
    const restored = await this.purchasesService.restorePurchases();
    if (restored) {
      this.router.navigateByUrl('/tabs', { replaceUrl: true });
    }
  }
}
```

For the actual purchase wiring (mapping `selectedPlan` to a `PurchasesPackage` from `getOfferings()`), see [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md).
