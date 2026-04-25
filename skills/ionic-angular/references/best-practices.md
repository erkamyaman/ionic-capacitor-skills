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
<ion-header>
  <ion-toolbar>
    <ion-title>{{ 'home.title' | translate }}</ion-title>
  </ion-toolbar>
</ion-header>
<ion-content>
  <!-- content -->
</ion-content>
```

```typescript
// home/home.page.ts
@Component({
  selector: 'app-home',
  standalone: true,
  imports: [IonContent, IonHeader, IonTitle, IonToolbar, TranslateModule],
  templateUrl: './home.page.html',
  styleUrls: ['./home.page.scss'],
})
export class HomePage {}
```

```scss
// home/home.page.scss — page-specific styles only
```

## Ionic component imports

Import the individual components you actually use — `IonContent`, `IonButton`, `IonHeader`, etc. **Never** `import { IonicModule } from '@ionic/angular'` in standalone components.

## Icons

Register icons explicitly via `addIcons()`:

```typescript
import { addIcons } from 'ionicons';
import { home, compass, settings } from 'ionicons/icons';

constructor() {
  addIcons({ home, compass, settings });
}
```

Then reference by name in templates:

```html
<ion-icon name="home"></ion-icon>
```

## Routing

- Lazy-load every page via `loadComponent` (or `loadChildren` for trees).
- Use functional `CanActivateFn` route guards, not class-based.

## State

- Prefer **Signals** for simple reactive state.
- Use **services** (`@Injectable({ providedIn: 'root' })`) for shared state across pages.
- Use `inject()` or constructor injection — both are fine; pick one and stick with it within a project.

## TypeScript

- `strict: true`, no `any`.
- Type Capacitor plugin responses explicitly.
