# Tabs Navigation (Angular)

`ion-tabs` + `ion-tab-bar`. Three tabs by default: Home, Explore, Settings. Show the AdMob banner on enter; hide on leave.

## Common ionicons

| Purpose       | Icon name      |
|---------------|---------------|
| Home          | `home`         |
| Explore       | `compass`      |
| Settings      | `settings`     |
| Profile       | `person`       |
| Search        | `search`       |
| Favorites     | `heart`        |
| Notifications | `notifications`|

## `tabs/tabs.page.html`

```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="home">
      <ion-icon name="home"></ion-icon>
      <ion-label>{{ 'tabs.home' | translate }}</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="explore">
      <ion-icon name="compass"></ion-icon>
      <ion-label>{{ 'tabs.explore' | translate }}</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="settings">
      <ion-icon name="settings"></ion-icon>
      <ion-label>{{ 'tabs.settings' | translate }}</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

## `tabs/tabs.page.ts`

```typescript
import { Component, OnDestroy, OnInit } from '@angular/core';
import {
  IonTabs, IonTabBar, IonTabButton, IonIcon, IonLabel,
} from '@ionic/angular/standalone';
import { addIcons } from 'ionicons';
import { home, compass, settings } from 'ionicons/icons';
import { TranslateModule } from '@ngx-translate/core';
import { AdsService } from '../services/ads.service';

@Component({
  selector: 'app-tabs',
  standalone: true,
  imports: [
    IonTabs, IonTabBar, IonTabButton, IonIcon, IonLabel, TranslateModule,
  ],
  templateUrl: './tabs.page.html',
  styleUrls: ['./tabs.page.scss'],
})
export class TabsPage implements OnInit, OnDestroy {
  constructor(private adsService: AdsService) {
    addIcons({ home, compass, settings });
  }

  async ngOnInit() {
    await this.adsService.showBanner();
  }

  async ngOnDestroy() {
    await this.adsService.hideBanner();
  }
}
```

## `tabs/tabs.page.scss`

```scss
// page-level styling for the tab bar / content
```

`AdsService.showBanner()` no-ops for premium users — the gate lives inside the service. See [services.md](services.md).
