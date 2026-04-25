# Tabs Navigation (Angular)

`ion-tabs` + `ion-tab-bar`. Three tabs by default: Home, Explore, Settings. Show the AdMob banner on enter; hide on leave.

## Common ionicons

The default scaffold uses `home`, `compass`, `settings` — registered via `addIcons` below. Other common picks if you customize the tabs: `person` (profile), `search`, `heart` (favorites), `notifications`. Browse the full set at <https://ionic.io/ionicons>.

## `tabs/tabs.page.html`

```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="home" [attr.aria-label]="'tabs.home' | translate">
      <ion-icon name="home" aria-hidden="true"></ion-icon>
      <ion-label>{{ 'tabs.home' | translate }}</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="explore" [attr.aria-label]="'tabs.explore' | translate">
      <ion-icon name="compass" aria-hidden="true"></ion-icon>
      <ion-label>{{ 'tabs.explore' | translate }}</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="settings" [attr.aria-label]="'tabs.settings' | translate">
      <ion-icon name="settings" aria-hidden="true"></ion-icon>
      <ion-label>{{ 'tabs.settings' | translate }}</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

`aria-hidden` on the icon prevents screen readers from announcing the icon name on top of the label. See [accessibility.md](accessibility.md).

## `tabs/tabs.page.ts`

```typescript
import {
  Component, ChangeDetectionStrategy, OnDestroy, OnInit, inject,
} from '@angular/core';
import {
  IonTabs, IonTabBar, IonTabButton, IonIcon, IonLabel,
} from '@ionic/angular/standalone';
import { TranslateModule } from '@ngx-translate/core';
import { AdsService } from '../services/ads.service';

@Component({
  selector: 'app-tabs',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    IonTabs, IonTabBar, IonTabButton, IonIcon, IonLabel, TranslateModule,
  ],
  templateUrl: './tabs.page.html',
  styleUrls: ['./tabs.page.scss'],
})
export class TabsPage implements OnInit, OnDestroy {
  private ads = inject(AdsService);

  async ngOnInit() {
    await this.ads.showBanner();
  }

  // Sync — Angular ignores returned Promises from ngOnDestroy, and a rejection
  // becomes an unhandled rejection on teardown. Fire-and-forget with a .catch
  // so transient hide-banner errors don't crash the surrounding navigation.
  ngOnDestroy() {
    this.ads.hideBanner().catch((err) => console.error('[Tabs] hideBanner', err));
  }
}
```

Icons are registered globally in `app.config.ts` via `addIcons({...})` (see [app-config.md](app-config.md)) — no per-component `addIcons` calls. This avoids the side-effecting-constructor smell and ensures icons are available before any component renders.

## `tabs/tabs.page.scss`

```scss
// page-level styling for the tab bar / content
```

`AdsService.showBanner()` no-ops for premium users — the gate lives inside the service. See [services.md](services.md).
