# RevenueCat (`@revenuecat/purchases-capacitor`)

In-app purchases / subscriptions. The paywall in this scaffold uses two packages: **weekly** (default) and **yearly** (with a discount badge).

## API keys — read from env, not hardcoded

> **Don't hardcode RevenueCat keys in source** — read them from build-time env vars. See [environments-and-keys.md](environments-and-keys.md). The public SDK key (`appl_…` / `goog_…`) is safe to ship in the bundle; the RevenueCat **REST API secret** is server-only — never put it in the app.

```typescript
// Vite (React / Vue) — pulled from .env files
const apiKey = Capacitor.getPlatform() === 'ios'
  ? import.meta.env.VITE_REVENUECAT_KEY_IOS
  : import.meta.env.VITE_REVENUECAT_KEY_ANDROID;

// Angular — pulled from src/environments/environment.ts
// import { environment } from '../environments/environment';
// const apiKey = Capacitor.getPlatform() === 'ios'
//   ? environment.revenueCat.iosKey
//   : environment.revenueCat.androidKey;
```

Tie the log level to the build mode so production builds don't emit DEBUG output:

```typescript
await Purchases.setLogLevel({
  level: import.meta.env.DEV ? LOG_LEVEL.DEBUG : LOG_LEVEL.WARN,
  // or, for Angular: environment.production ? LOG_LEVEL.WARN : LOG_LEVEL.DEBUG
});
```

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

  await Purchases.setLogLevel({
    level: import.meta.env.DEV ? LOG_LEVEL.DEBUG : LOG_LEVEL.WARN,
  });

  // ❌ DO NOT hardcode the keys — these placeholders are for illustration only.
  //    Replace with build-time env vars per the section above.
  const apiKey = Capacitor.getPlatform() === 'ios'
    ? import.meta.env.VITE_REVENUECAT_KEY_IOS
    : import.meta.env.VITE_REVENUECAT_KEY_ANDROID;

  await Purchases.configure({ apiKey });
  purchasesInitialized = true;
}

export async function isPremiumUser(): Promise<boolean> {
  if (!purchasesInitialized) return false;
  try {
    const { customerInfo } = await Purchases.getCustomerInfo();
    // "Any active entitlement" — fine for single-tier apps. For multi-tier
    // (e.g. 'pro' vs 'team'), check the named entitlement explicitly:
    //   return customerInfo.entitlements.active['pro'] != null;
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
  if (!purchasesInitialized) return false;
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
  if (!purchasesInitialized) return false;
  try {
    const { customerInfo } = await Purchases.restorePurchases();
    return Object.keys(customerInfo.entitlements.active).length > 0;
  } catch {
    return false;
  }
}
```

`getOfferings()` flattens RevenueCat's `PurchasesOfferings` shape down to just the current offering's `availablePackages` — that's what the paywall needs. If you want richer behavior (named offerings, A/B testing different price tiers), expose the full `offerings` object instead.

## React to entitlement changes (premium expires mid-session)

Without a listener, `isPremiumUser()` returns whatever was true at the last `getCustomerInfo()` call. If a subscription expires while the app is open, the UI keeps showing premium features until next launch. RevenueCat exposes a push channel for this:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

await Purchases.addCustomerInfoUpdateListener((customerInfo) => {
  const isPremium = Object.keys(customerInfo.entitlements.active).length > 0;
  // Push the new value into your framework's reactive store:
  //   Angular: inject PurchasesService and update its signal
  //   React:   call your store's set() function
  //   Vue:     write to the module-level ref
});
```

Wire this once at startup (inside `initializePurchases()` or right after) and the UI propagates entitlement flips in real time — both grants and revocations.

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

> **Display the price from the SDK, not hardcoded copy.** Each `PurchasesPackage` exposes `pkg.product.priceString` — a localized, currency-formatted string from the App Store / Play Store. Hardcoding `$4.99/week` ships a wrong price to every non-USD locale and the App Store scrutinizes mismatches.

## App Store / Play Store requirements

- The Restore Purchases button is **required** by Apple — App Review will reject builds that omit it.
- iOS sandbox testing requires a Sandbox tester account configured in App Store Connect.
- Android testing requires a closed-testing track and a test license account in Play Console.
