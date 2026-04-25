# Built-in Control Flow (`@if` / `@for` / `@switch`)

Modern Angular's built-in control flow is **the default** — `*ngIf` / `*ngFor` / `*ngSwitch` are legacy and slated for removal. Use the new `@`-syntax for every new template, and migrate when you touch existing ones.

## `@if` / `@else if` / `@else`

```html
<!-- ❌ Legacy -->
<ion-item *ngIf="!isPremium" button (click)="removeAds()">…</ion-item>

<!-- ✅ Modern -->
@if (!isPremium()) {
  <ion-item button (click)="removeAds()">…</ion-item>
}
```

With `else` branches:

```html
@if (loading()) {
  <ion-spinner />
} @else if (error()) {
  <p>Couldn't load.</p>
} @else {
  <article>{{ data() }}</article>
}
```

No structural-directive imports needed (no `NgIf` import to remove).

## `@for` with `track`

```html
<!-- ❌ Legacy -->
<div
  *ngFor="let option of subscriptionOptions; trackBy: trackById"
  [class.selected]="selectedPlan === option.id"
>…</div>

<!-- ✅ Modern -->
@for (option of subscriptionOptions; track option.id) {
  <div [class.selected]="selectedPlan() === option.id">…</div>
} @empty {
  <p>No options.</p>
}
```

`track` is **required** — it's the mechanism Angular uses to identify items across renders. Without it, the compiler errors. Common `track` values:

| Value | Use when |
|-------|----------|
| `track item.id` | Items have stable IDs (most common). |
| `track $index` | Order is the identity (rare; only when items are interchangeable). |
| `track item` | Identity by reference. Fine for primitives in a non-mutating array. |

`@empty` renders when the iterable has zero items — replaces the old `*ngIf="items.length === 0"` companion block.

`@for` also exposes contextual variables:

```html
@for (item of items; track item.id; let i = $index, isFirst = $first, isLast = $last) {
  <div [class.first]="isFirst" [class.last]="isLast">{{ i + 1 }}. {{ item.name }}</div>
}
```

## `@switch`

```html
@switch (status()) {
  @case ('loading') { <ion-spinner /> }
  @case ('error')   { <p>Failed.</p> }
  @case ('ready')   { <article>{{ data() }}</article> }
  @default          { <p>Unknown state.</p> }
}
```

`@switch` does strict equality (`===`). For more complex predicates, use `@if` / `@else if` chains.

## Migration

Angular ships an automated migration:

```bash
npx ng generate @angular/core:control-flow
```

It rewrites `*ngIf` / `*ngFor` / `*ngSwitch` → `@if` / `@for` / `@switch` across the project. Always commit before running, then review the diff (it's usually clean but occasionally needs touch-ups for complex `*ngIf="x as y"` aliases).

## Drop the structural-directive imports

After migration, remove `NgIf`, `NgFor`, `NgSwitch` (and `NgSwitchCase`, `NgSwitchDefault`) from your component's `imports`. They're no longer used and bloat the bundle.

```typescript
// ❌ Before
imports: [IonContent, IonButton, NgFor, NgIf, NgClass, TranslateModule]

// ✅ After
imports: [IonContent, IonButton, NgClass, TranslateModule]
```

`NgClass` and `NgStyle` aren't replaced by built-in control flow — keep those if used.

## In this scaffold

- **Paywall page**: the option-card list uses `@for (option of subscriptionOptions; track option.id)`.
- **Settings page**: every conditional row uses `@if (!isPremium())`.
- **Onboarding**: slide list uses `@for`.

See [paywall-page.md](paywall-page.md) and [settings-page.md](settings-page.md) for the full updated templates.
