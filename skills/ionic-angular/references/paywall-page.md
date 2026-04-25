# Paywall Page (Angular)

Shown immediately after onboarding. Two RevenueCat subscription packages (weekly default, yearly with 50% OFF badge) plus a Restore Purchases button.

## Required actions

- **Subscribe** — purchase the selected package via RevenueCat.
- **Continue with ads** — skip to `/tabs` without buying.
- **Restore Purchases** — required by Apple. Restores any prior purchase for the signed-in store account.

## `paywall/paywall.page.html`

Renders from `offerings()` (loaded in `ngOnInit`) using built-in control flow (`@if` / `@for` — see [control-flow.md](control-flow.md)). The cards form a `radiogroup` for accessibility — see [accessibility.md](accessibility.md).

```html
<ion-page>
  <ion-content [fullscreen]="true">
    <div class="paywall-container">
      <h1>{{ 'paywall.title' | translate }}</h1>

      @if (loading()) {
        <ion-spinner />
      } @else if (loadError()) {
        <ion-text color="danger">
          <p>{{ 'paywall.loadError' | translate }}</p>
        </ion-text>
        <ion-button fill="outline" (click)="loadOfferings()">
          {{ 'paywall.retry' | translate }}
        </ion-button>
      } @else {
        <div
          class="subscription-options"
          role="radiogroup"
          [attr.aria-label]="'paywall.title' | translate"
        >
          @for (pkg of offerings(); track pkg.identifier) {
            <button
              type="button"
              class="option-card"
              role="radio"
              [attr.aria-checked]="selectedId() === pkg.identifier"
              [class.selected]="selectedId() === pkg.identifier"
              (click)="selectedId.set(pkg.identifier)"
            >
              <h3>{{ pkg.product.title }}</h3>
              <p>{{ pkg.product.priceString }}</p>
            </button>
          } @empty {
            <ion-text color="medium">
              <p>{{ 'paywall.noOfferings' | translate }}</p>
            </ion-text>
          }
        </div>
      }

      <ion-button expand="block"
        [disabled]="!selectedId() || subscribing()"
        (click)="subscribe()">
        @if (subscribing()) { <ion-spinner name="dots" /> }
        @else { {{ 'paywall.subscribe' | translate }} }
      </ion-button>

      <ion-button fill="clear" (click)="skip()">
        {{ 'paywall.skip' | translate }}
      </ion-button>

      <ion-button fill="clear" size="small" (click)="restore()">
        {{ 'paywall.restore' | translate }}
      </ion-button>
    </div>
  </ion-content>
</ion-page>
```

Three error-aware additions:

- `@else if (loadError())` branch with a Retry button when `getOfferings()` fails.
- `@empty` companion for the `@for` so a zero-package response (RevenueCat misconfigured, store offline) shows a message instead of a blank screen.
- `subscribing()` signal disables the Subscribe button + shows a spinner so the user can't double-tap mid-purchase.

`pkg.product.priceString` is the localized, currency-formatted string the store will charge. **Don't** hardcode `$4.99/week` — it's wrong in every non-USD locale and Apple scrutinizes mismatches.

## `paywall/paywall.page.scss`

```scss
.paywall-container { /* layout / spacing */ }

.subscription-options { /* grid of option cards */ }

.option-card {
  min-height: 48px;       // a11y touch target — 48 dp Material / >= 44 pt iOS
  padding: 16px;
  background: none;
  border: 2px solid var(--ion-color-step-200);
  border-radius: 12px;
  text-align: left;
  cursor: pointer;

  &.selected {
    border-color: var(--ion-color-primary);
  }

  &:focus-visible {
    outline: 2px solid var(--ion-color-primary);
    outline-offset: 2px;
  }
}
```

## `paywall/paywall.page.ts`

```typescript
import {
  Component, ChangeDetectionStrategy, OnInit, inject, signal,
} from '@angular/core';
import { Router } from '@angular/router';
import {
  IonContent, IonPage, IonButton, IonSpinner, IonText, ToastController,
} from '@ionic/angular/standalone';
import { TranslateModule, TranslateService } from '@ngx-translate/core';
import { PurchasesPackage } from '@revenuecat/purchases-capacitor';
import { PurchasesService } from '../services/purchases.service';

@Component({
  selector: 'app-paywall',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    IonContent, IonPage, IonButton, IonSpinner, IonText,
    TranslateModule,
  ],
  templateUrl: './paywall.page.html',
  styleUrls: ['./paywall.page.scss'],
})
export class PaywallPage implements OnInit {
  private router = inject(Router);
  private purchases = inject(PurchasesService);
  private toast = inject(ToastController);
  private translate = inject(TranslateService);

  // Private writable signals — exposed as readonly via the .asReadonly() pattern
  // would be cleaner, but for a single-component scope we keep them direct.
  // Just don't mark them `readonly` — they ARE writable (we call .set() in ngOnInit).
  readonly offerings = signal<PurchasesPackage[]>([]);
  readonly loading = signal(true);
  readonly loadError = signal(false);
  readonly selectedId = signal<string | null>(null);
  readonly subscribing = signal(false);

  async ngOnInit() {
    await this.loadOfferings();
  }

  async loadOfferings() {
    this.loading.set(true);
    this.loadError.set(false);
    try {
      const pkgs = await this.purchases.getOfferings();
      this.offerings.set(pkgs);
      // Default-select the first package (typically the weekly tier — order is from RevenueCat).
      if (pkgs.length > 0 && this.selectedId() === null) {
        this.selectedId.set(pkgs[0].identifier);
      }
    } catch (err) {
      console.error('[Paywall] getOfferings failed', err);
      this.loadError.set(true);
    } finally {
      this.loading.set(false);
    }
  }

  async subscribe() {
    const pkg = this.offerings().find(p => p.identifier === this.selectedId());
    if (!pkg || this.subscribing()) return;

    this.subscribing.set(true);
    try {
      const ok = await this.purchases.purchase(pkg);
      if (ok) {
        this.router.navigateByUrl('/tabs', { replaceUrl: true });
      }
      // ok === false on user cancel — silent, no error toast.
    } catch (err) {
      console.error('[Paywall] purchase failed', err);
      await this.showToast('paywall.purchaseError', 'danger');
    } finally {
      this.subscribing.set(false);
    }
  }

  skip() {
    this.router.navigateByUrl('/tabs', { replaceUrl: true });
  }

  async restore() {
    try {
      const ok = await this.purchases.restore();
      if (ok) {
        this.router.navigateByUrl('/tabs', { replaceUrl: true });
      } else {
        await this.showToast('paywall.restoreEmpty', 'medium');
      }
    } catch (err) {
      console.error('[Paywall] restore failed', err);
      await this.showToast('paywall.restoreError', 'danger');
    }
  }

  private async showToast(key: string, color: 'danger' | 'medium') {
    const toast = await this.toast.create({
      message: this.translate.instant(key),
      duration: 2500,
      color,
      position: 'bottom',
    });
    await toast.present();
  }
}
```

The error / pending UX is what separates an example from a shippable paywall:

- **`loadError`** — `getOfferings()` rejection no longer disappears silently. The retry button calls `loadOfferings()` again.
- **`subscribing`** — disables the Subscribe button during the in-flight purchase + shows a spinner. Prevents double-tap purchases.
- **`purchase()` returning `false`** is silent (user cancelled — Apple doesn't want us to nag). A **thrown** error gets a toast.
- **`restore()` returning `false`** shows a "no purchases to restore" toast — Apple specifically requires the restore flow to give the user feedback.

Notes:

- **`ChangeDetectionStrategy.OnPush`** — with `selectedId` / `offerings` / `loading` as signals, Angular only re-renders this component when one of them changes.
- **`inject()`** for all DI; no constructor parameters.
- **No `NgFor` / `NgIf` / `NgClass`** in `imports` — built-in `@for` / `@if` doesn't need them.
- **`<ion-page>`** wraps the content for proper Ionic transitions.
- **`PurchasesService.purchase()`** calls `refresh()` internally on success — the rest of the app (Settings "Remove Ads", banner-hide on Tabs) updates reactively via the signal.

## RevenueCat package shape

`PurchasesPackage` exposes:

- `identifier` — string ID (e.g., `'$rc_weekly'`, `'$rc_annual'`).
- `product.title` — localized product title from the store listing.
- `product.priceString` — localized, currency-formatted price.
- `product.price` — numeric price (rare to need; prefer `priceString` for display).

If you want a discount-percentage badge ("50% OFF"), compute it from the package list rather than hardcoding — the discount changes per offering / locale.

For the underlying SDK API (`getOfferings`, `purchasePackage`, error codes), see [`../../ionic-shared/references/revenuecat.md`](../../ionic-shared/references/revenuecat.md).
