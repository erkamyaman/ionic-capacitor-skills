# Signals (Angular)

**Prefer Signals for reactive state.** Services in this scaffold should expose `Signal<T>` and `WritableSignal<T>` rather than plain values, so templates re-render automatically when the underlying state changes.

> Stick to the **stable** signal APIs documented here (`signal`, `computed`, `effect`, `untracked`, `input`, `output`, `toSignal`, `toObservable`). Avoid Angular APIs marked "developer preview" / "experimental" — `linkedSignal`, `resource`, `httpResource`, `model`, Signal Forms, `afterRenderEffect` etc. They may be useful but their shape and behavior can change between minor versions. Wait for them to graduate before relying on them in production.

## When to use what (stable APIs)

| API              | Use for |
|------------------|---------|
| `signal(initial)`| Mutable local state. |
| `computed(fn)`   | Derived state — recomputed lazily when its dependencies change. |
| `effect(fn)`     | **Side effects only** (logging, third-party DOM, manual subscriptions). Avoid for state mutations. |
| `untracked(fn)`  | Read a signal inside an `effect` / `computed` without registering it as a dependency. |
| `input()` / `input.required()` | Signal-based component inputs. |
| `output()`       | Signal-based component outputs (replaces `EventEmitter`). |
| `toSignal(obs$)` | Convert an Observable into a signal. |
| `toObservable(s)`| Convert a signal into an Observable. |

## Pattern: services as signal stores

The full set of services in [services.md](services.md) follows this pattern. Sketch:

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

Components bind to `purchases.isPremium()` directly — no `OnInit` + `await isPremium()` plumbing.

## Pattern: a legitimate `effect()` — pushing signal state to a non-Angular system

`effect()` is for pushing signal state into systems that aren't reactive themselves: the DOM, a third-party library, the console, an analytics tracker. The example below is **illustrative** — it demonstrates the shape of a "DOM bridge" effect.

```typescript
import { Injectable, signal, Signal, effect } from '@angular/core';
import { applyTheme, ThemeMode } from '../utils/theme';

@Injectable({ providedIn: 'root' })
export class IllustrativeThemeService {
  private readonly _mode = signal<ThemeMode>('system');
  readonly mode: Signal<ThemeMode> = this._mode.asReadonly();

  constructor() {
    // Bridges a signal value into a non-Angular system (the <html> class).
    effect(() => applyTheme(this._mode()));
  }
}
```

**Don't** use `effect()` to write to other signals — use `computed()` for derived values, or update both from the same method.

> **Heads-up:** in this scaffold, the actual `ThemeService` ([services.md](services.md)) does **not** use `effect()` — calling `applyTheme()` directly from `initialize()` / `setMode()` avoids a cold-start flash when the constructor effect would fire once with the default `'system'` before the saved value is loaded. The pattern above is the right teaching example; the production code is the simpler explicit-call version.

## Pattern: derived state with `computed()`

```typescript
import { signal, computed } from '@angular/core';

const count = signal(0);
const doubled = computed(() => count() * 2);
const isEven = computed(() => count() % 2 === 0);

count.set(3);
console.log(doubled());   // 6
console.log(isEven());    // false
```

`computed()` only re-runs when one of its read signals changes, and only when something actually reads its value (lazy). Free memoization.

## Pattern: `untracked()` to break a dependency

Inside an `effect` or `computed`, every signal read becomes a dependency. To read without subscribing:

```typescript
import { signal, effect, untracked } from '@angular/core';

const search = signal('');
const lastQuery = signal('');

effect(() => {
  const q = search();                     // tracked — re-runs when search changes
  const previous = untracked(lastQuery);  // NOT tracked — read once, no subscription
  console.log(`new: ${q}, was: ${previous}`);
  lastQuery.set(q);                       // OK — not creating a cycle
});
```

Without `untracked`, reading `lastQuery()` would make the effect run twice per change (once for `search`, once when `lastQuery` updates).

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

- For one-shot async values that complete (use a Promise / `async-await` instead, then `.set()` the result on a signal if the UI needs it).
- Inside `effect()` to mutate other signals (creates implicit cycles — use `computed()`).
- For form state — Reactive Forms (`FormGroup` / `FormControl`) for anything beyond a couple of fields; **Template-driven forms** (`ngModel`) are still fine for very small forms or quick prototypes. Template-driven needs `FormsModule` in the standalone component's `imports` array — easy to forget when copy-pasting an `[(ngModel)]="..."` snippet. (**Signal Forms** is the Angular team's in-development signal-native form API exposed via `@angular/forms/signals` — but it's marked developer-preview / experimental, so the stable shape may change between minor versions. Wait for it to graduate before adopting in production.)

## Migration nudge

Old `BehaviorSubject<T>` patterns can almost always be replaced with `signal<T>(initial)` + `.asReadonly()`. Both give you "current value + change notification", but signals integrate with change detection without manual `async` pipes.
