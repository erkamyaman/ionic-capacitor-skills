# RevenueCat (`@revenuecat/purchases-capacitor`)

In-app purchases / subscriptions. The paywall in this scaffold uses two packages: **weekly** (default) and **yearly** (with a discount badge).

## Utility (framework-agnostic)

```typescript
// utils/purchases.ts
import { Capacitor } from '@capacitor/core';
import {
  Purchases,
  LOG_LEVEL,
  PURCHASES_ERROR_CODE,
  PurchasesPackage,
} from '@revenuecat/purchases-capacitor';

let purchasesInitialized = false;

export async function initializePurchases(): Promise<void> {
  if (!Capacitor.isNativePlatform()) return;

  await Purchases.setLogLevel({ level: LOG_LEVEL.DEBUG }); // use WARN in production

  const apiKey = Capacitor.getPlatform() === 'ios'
    ? 'appl_YOUR_IOS_API_KEY'
    : 'goog_YOUR_ANDROID_API_KEY';

  await Purchases.configure({ apiKey });
  purchasesInitialized = true;
}

export async function isPremiumUser(): Promise<boolean> {
  if (!purchasesInitialized) return false;
  try {
    const { customerInfo } = await Purchases.getCustomerInfo();
    return Object.keys(customerInfo.entitlements.active).length > 0;
  } catch {
    return false;
  }
}

export async function getOfferings(): Promise<PurchasesPackage[]> {
  if (!purchasesInitialized) return [];
  try {
    const { offerings } = await Purchases.getOfferings();
    return offerings?.current?.availablePackages ?? [];
  } catch {
    return [];
  }
}

export async function purchasePackage(pkg: PurchasesPackage): Promise<boolean> {
  try {
    const { customerInfo } = await Purchases.purchasePackage({ aPackage: pkg });
    return Object.keys(customerInfo.entitlements.active).length > 0;
  } catch (error: unknown) {
    if ((error as { code?: string })?.code === PURCHASES_ERROR_CODE.PURCHASE_CANCELLED_ERROR) {
      return false;
    }
    throw error;
  }
}

export async function restorePurchases(): Promise<boolean> {
  try {
    const { customerInfo } = await Purchases.restorePurchases();
    return Object.keys(customerInfo.entitlements.active).length > 0;
  } catch {
    return false;
  }
}
```

## Paywall flow

```
Onboarding → Paywall → Tabs (main app)
```

The paywall MUST appear immediately after onboarding completes, with three actions:

1. **Subscribe** — calls `purchasePackage` for the selected plan.
2. **Continue with ads** — skips to `/tabs` without purchasing.
3. **Restore Purchases** — calls `restorePurchases`; navigates to `/tabs` on success.

Two packages, weekly default-selected:

| ID       | Title          | Notes |
|----------|----------------|-------|
| `weekly` | Weekly         | Default selection |
| `yearly` | Yearly (50% OFF) | Highlighted with a badge |

## API keys

The placeholders `appl_YOUR_IOS_API_KEY` / `goog_YOUR_ANDROID_API_KEY` must be replaced with real keys from the RevenueCat dashboard before any test purchase will work.

## App Store / Play Store requirements

- The Restore Purchases button is **required** by Apple — App Review will reject builds that omit it.
- iOS sandbox testing requires a Sandbox tester account configured in App Store Connect.
- Android testing requires a closed-testing track and a test license account in Play Console.
