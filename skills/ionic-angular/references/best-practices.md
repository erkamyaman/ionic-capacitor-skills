# Angular Best Practices

## Standalone components, separate files

❌ NgModules + `IonicModule`:

```typescript
// home.module.ts
@NgModule({
  imports: [CommonModule, IonicModule],
  declarations: [HomePage],
})
export class HomePageModule {}
```

❌ Inline `template` / `styles`:

```typescript
@Component({
  selector: 'app-home',
  standalone: true,
  imports: [IonContent, IonHeader, IonTitle, IonToolbar, TranslateModule],
  template: `
    <ion-header>
      <ion-toolbar>
        <ion-title>{{ 'home.title' | translate }}</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content><!-- ... --></ion-content>
  `,
})
export class HomePage {}
```

✅ Separate `.html`, `.ts`, `.scss`:

```html
<!-- home/home.page.html -->
<ion-page>
  <ion-header>
    <ion-toolbar>
      <ion-title>{{ 'home.title' | translate }}</ion-title>
    </ion-toolbar>
  </ion-header>
  <ion-content>
    <!-- content -->
  </ion-content>
</ion-page>
```

```typescript
// home/home.page.ts
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-home',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [IonContent, IonPage, IonHeader, IonTitle, IonToolbar, TranslateModule],
  templateUrl: './home.page.html',
  styleUrls: ['./home.page.scss'],
})
export class HomePage {}
```

```scss
// home/home.page.scss — page-specific styles only
```

## Always use `OnPush`

Set `changeDetection: ChangeDetectionStrategy.OnPush` on every component in the scaffold. With signals as the state primitive, `OnPush` is effectively free — Angular only re-renders when a signal read in the template changes. Default change detection re-runs on every event in the zone, which is wasteful and unpredictable.

```typescript
@Component({
  // ...
  changeDetection: ChangeDetectionStrategy.OnPush,
})
```

## Modern control flow

Use `@if` / `@for` / `@switch` — never `*ngIf` / `*ngFor` / `*ngSwitch`. See [control-flow.md](control-flow.md).

```html
@for (option of subscriptionOptions; track option.id) {
  <button class="option-card" …>…</button>
}

@if (!isPremium()) {
  <ion-item button (click)="removeAds()">…</ion-item>
}
```

## Ionic component imports

Import the individual components you actually use — `IonContent`, `IonButton`, `IonHeader`, etc. **Never** `import { IonicModule } from '@ionic/angular'` in standalone components.

Always wrap pages in `<ion-page>` for proper Ionic transitions and lifecycle.

## Icons

Register icons explicitly via `addIcons()`:

```typescript
import { addIcons } from 'ionicons';
import { home, compass, settings } from 'ionicons/icons';

constructor() {
  addIcons({ home, compass, settings });
}
```

Then reference by name in templates, with `aria-hidden` (the icon is decorative — the adjacent label carries the meaning):

```html
<ion-icon name="home" aria-hidden="true"></ion-icon>
```

See [accessibility.md](accessibility.md) for more.

## Routing

- Lazy-load every page via `loadComponent` (or `loadChildren` for trees).
- Use functional `CanMatchFn` (preferred for lazy-route gates) or `CanActivateFn` route guards, not class-based — see [onboarding-guard.md](onboarding-guard.md).
- Configure the router with `withViewTransitions()` and `withComponentInputBinding()` — see [app-config.md](app-config.md).

## State

- Prefer **Signals** for reactive state — `signal()`, `computed()`, `effect()`. See [signals.md](signals.md).
- Use **services** (`@Injectable({ providedIn: 'root' })`) for shared state across pages, exposing it as readonly signals — see [services.md](services.md).
- Avoid Angular APIs marked "developer preview" / "experimental" (e.g. `linkedSignal`, `resource`, Signal Forms) until they graduate to stable.

## DI: `inject()` over constructor injection

Use `inject()` for services and tokens. Reserve constructor parameters for cases that genuinely need them (rare in modern Angular).

```typescript
// ✅ Preferred
export class PaywallPage {
  private router = inject(Router);
  private purchases = inject(PurchasesService);
}

// ⚠️ Older idiom — works, but not the canonical style for new code
export class PaywallPage {
  constructor(
    private router: Router,
    private purchases: PurchasesService,
  ) {}
}
```

`inject()` is also the only DI mechanism inside functional guards / resolvers / `effect()` callbacks — using one consistent style across the project keeps things uniform.

## Performance

- `@defer` only chunks that are genuinely heavy (Swiper, charting libs, rich-text editors). Tiny subtrees aren't worth the HTTP roundtrip — see [defer-loading.md](defer-loading.md).
- Lazy-load every route.
- Trust signals + `OnPush` to do the right thing — don't reach for `markForCheck()` / `detectChanges()`.

## App lifecycle wiring

Mobile apps spend a lot of time in the background. Wire the standard `@capacitor/app` listeners early (in `APP_INITIALIZER` or a dedicated `AppLifecycleService`):

```typescript
import { App } from '@capacitor/app';

App.addListener('appStateChange', ({ isActive }) => {
  if (isActive) {
    // App returning to foreground — refresh stale state.
    // e.g., purchases.refresh() to re-check subscription status.
  }
});

App.addListener('backButton', ({ canGoBack }) => {
  // Android hardware back button — let Ionic handle navigation by default.
  // Override here only for explicit confirmations.
});
```

Particularly important: call `purchases.refresh()` on `appStateChange`'s `isActive: true` so a subscription that lapsed while backgrounded propagates to the UI on resume. (The `addCustomerInfoUpdateListener` in `PurchasesService.initialize()` covers most cases, but a foreground-event refresh is belt-and-suspenders.)

## Error handling

Provide a custom `ErrorHandler` (see [app-config.md](app-config.md)) so unhandled exceptions land somewhere meaningful — at minimum the console with context, ideally a crash reporter (see [`../../ionic-sentry/`](../../ionic-sentry/SKILL.md)).

For user-facing failure paths (purchase failed, network down, restore returned no purchases), use Ionic's `ToastController` rather than letting the error vanish silently. The paywall page in [paywall-page.md](paywall-page.md) shows the pattern — `inject(ToastController)`, then `await toast.create({...}).present()`.

## State management at scale

`providedIn: 'root'` for every service is fine for the ~5 services in this scaffold but eats tree-shaking at scale (50+ services). When you cross ~10 services or split the app into clearly-separated feature areas:

- Group services in `src/app/&lt;feature&gt;/` folders, not one flat `services/` folder.
- Use `providers: [...]` on lazy routes for feature-scoped services (so they only initialize when the feature loads).
- Keep `providedIn: 'root'` for genuinely-global state (theme, auth, premium status).

## TypeScript

- `strict: true`, no `any`.
- Type Capacitor plugin responses explicitly.
- Type `(ionChange)` / `(ionInput)` payloads as `CustomEvent<{ value: T }>` rather than bare `CustomEvent`.

## Snippet caveat — education vs production

The code samples in this skill prioritize clarity. They show the canonical patterns (signals, `OnPush`, `inject()`, `@for` / `@if`) but cut a few corners that a production build needs to handle:

- **Async error handling.** Most `try { ... } finally { ... }` blocks log and swallow. Production should toast / report (see [paywall-page.md](paywall-page.md) for the full pattern).
- **Pending state.** Long-running calls (purchase, restore) need a "subscribing" signal that disables the trigger button + shows a spinner.
- **Network offline.** Snippets assume Capacitor calls succeed. Wrap network-dependent paths in a check (or rely on the `@capacitor/network` listener pattern in [`../../ionic-native-essentials/references/network.md`](../../ionic-native-essentials/references/network.md)).
- **RTL / long translations.** Layouts assume English-length copy. Verify with German, Turkish, Arabic before shipping.
- **Stale state on resume.** Use the `appStateChange` listener (above) to refresh entitlements / fetch fresh data when the app returns to foreground.

Treat the snippets as the correct shape, not the complete shipping path.

## Angular MCP

If the project's environment exposes an Angular MCP server (`ng mcp` or via Angular CLI), prefer its `get_best_practices` tool for the most up-to-date guidance — its content is versioned to the project's Angular version. Fall back to this skill when MCP isn't available.
