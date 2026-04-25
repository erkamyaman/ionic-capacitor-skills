# Signals (Angular)

**Prefer Signals for reactive state.** Services in this scaffold should expose `Signal<T>` and `WritableSignal<T>` rather than plain values, so templates re-render automatically when the underlying state changes.

## When to use what

| API              | Use for |
|------------------|---------|
| `signal(initial)`| Mutable local state. |
| `computed(fn)`   | Derived state — recomputed lazily when its dependencies change. |
| `linkedSignal(fn)` | Writable state derived from a source signal — resets when the source changes. |
| `resource(...)`  | Async data fetched via a signal — exposes `.value()`, `.status()`, `.error()`. |
| `effect(fn)`     | **Side effects only** (logging, third-party DOM, manual subscriptions). Avoid for state mutations. |
| `toSignal(obs$)` | Convert an Observable into a signal. |
| `toObservable(s)`| Convert a signal into an Observable. |

## Pattern: services as signal stores

The bare wrappers in [services.md](services.md) work, but the production-grade pattern is to expose state as readonly signals so components can bind directly:

```typescript
// services/purchases.service.ts
import { Injectable, signal, Signal } from '@angular/core';
import {
  initializePurchases,
  isPremiumUser,
  getOfferings,
  purchasePackage,
  restorePurchases,
} from '../utils/purchases';
import { PurchasesPackage } from '@revenuecat/purchases-capacitor';

@Injectable({ providedIn: 'root' })
export class PurchasesService {
  private readonly _isPremium = signal(false);
  readonly isPremium: Signal<boolean> = this._isPremium.asReadonly();

  async initialize(): Promise<void> {
    await initializePurchases();
    await this.refresh();
  }

  async refresh(): Promise<void> {
    this._isPremium.set(await isPremiumUser());
  }

  async purchase(pkg: PurchasesPackage): Promise<boolean> {
    const ok = await purchasePackage(pkg);
    if (ok) await this.refresh();
    return ok;
  }

  async restore(): Promise<boolean> {
    const ok = await restorePurchases();
    if (ok) await this.refresh();
    return ok;
  }

  getOfferings = getOfferings;
}
```

Now any component can bind to `purchases.isPremium()` in its template and the UI updates automatically when entitlements change. The `Settings` page no longer needs `OnInit` + `await isPremium()` — it just reads the signal.

## Pattern: theme as a signal

```typescript
// services/theme.service.ts
import { Injectable, signal, Signal, effect } from '@angular/core';
import { getTheme, setTheme as persistTheme, applyTheme, ThemeMode } from '../utils/theme';

@Injectable({ providedIn: 'root' })
export class ThemeService {
  private readonly _mode = signal<ThemeMode>('system');
  readonly mode: Signal<ThemeMode> = this._mode.asReadonly();

  constructor() {
    // Apply theme to <html> whenever the mode signal changes.
    effect(() => applyTheme(this._mode()));
  }

  async initialize(): Promise<void> {
    this._mode.set(await getTheme());
  }

  async setMode(mode: ThemeMode): Promise<void> {
    this._mode.set(mode);
    await persistTheme(mode);
  }
}
```

The `effect()` here is the legitimate use — it pushes signal state into a non-Angular system (the DOM class on `<html>`). Don't use `effect()` to mutate other signals — use `computed()` or `linkedSignal()` instead.

## Pattern: signal inputs and outputs

For new components, prefer the signal-based forms:

```typescript
import { Component, input, output } from '@angular/core';

@Component({ /* ... */ })
export class PaywallOptionCard {
  // Signal input — required + typed
  readonly option = input.required<{ id: string; title: string; price: string; badge: string | null }>();
  readonly selected = input(false);

  // Signal output
  readonly select = output<string>();

  onClick() {
    this.select.emit(this.option().id);
  }
}
```

Template usage: `<app-paywall-option-card [option]="opt" [selected]="opt.id === selectedPlan()" (select)="selectedPlan.set($event)" />`.

## When NOT to use signals

- For one-shot async values that complete (use a Promise / `async-await` instead).
- Inside `effect()` to mutate other signals (creates implicit cycles — use `computed()`).
- For form state if you're using Reactive Forms — `FormControl.valueChanges` is fine. Signal Forms (in recent Angular) is the new path; check the project's Angular version before adopting.

## Migration nudge

Old `BehaviorSubject<T>` patterns can almost always be replaced with `signal<T>(initial)` + `.asReadonly()`. Both give you "current value + change notification", but signals integrate with change detection without manual `async` pipes.
