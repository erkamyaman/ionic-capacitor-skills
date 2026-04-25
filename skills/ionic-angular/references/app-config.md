# App Configuration (Angular)

## `app.config.ts`

```typescript
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter, RouteReuseStrategy } from '@angular/router';
import { provideIonicAngular, IonicRouteStrategy } from '@ionic/angular/standalone';
import { provideHttpClient, HttpClient } from '@angular/common/http';
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { routes } from './app.routes';

export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideIonicAngular({ mode: 'md' }),
    provideHttpClient(),
    { provide: RouteReuseStrategy, useClass: IonicRouteStrategy },
    importProvidersFrom(
      TranslateModule.forRoot({
        defaultLanguage: 'en',
        loader: {
          provide: TranslateLoader,
          useFactory: createTranslateLoader,
          deps: [HttpClient],
        },
      }),
    ),
  ],
};
```

## `app.component.html`

```html
<ion-app>
  <ion-router-outlet></ion-router-outlet>
</ion-app>
```

## `app.component.ts`

```typescript
import { Component, OnInit } from '@angular/core';
import { IonApp, IonRouterOutlet } from '@ionic/angular/standalone';
import { AdsService } from './services/ads.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [IonApp, IonRouterOutlet],
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit {
  constructor(private adsService: AdsService) {}

  async ngOnInit() {
    await this.adsService.initialize();
  }
}
```

`AdsService.initialize()` calls `initializeAdMob()` from `utils/admob.ts` — see [`../../ionic-shared/references/admob.md`](../../ionic-shared/references/admob.md).

## `main.ts`

Standard Ionic + Angular bootstrap with the dark palette CSS imported so theme switching works:

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

import '@ionic/angular/css/core.css';
import '@ionic/angular/css/normalize.css';
import '@ionic/angular/css/structure.css';
import '@ionic/angular/css/typography.css';
import '@ionic/angular/css/padding.css';
import '@ionic/angular/css/float-elements.css';
import '@ionic/angular/css/text-alignment.css';
import '@ionic/angular/css/text-transformation.css';
import '@ionic/angular/css/flex-utils.css';
import '@ionic/angular/css/display.css';
import '@ionic/angular/css/palettes/dark.class.css';

import './theme/variables.scss';

bootstrapApplication(AppComponent, appConfig);
```
