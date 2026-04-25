# App Configuration (Angular)

## `app.config.ts`

```typescript
import {
  ApplicationConfig,
  APP_INITIALIZER,
  ErrorHandler,
  importProvidersFrom,
  inject,
} from '@angular/core';
import {
  provideRouter,
  RouteReuseStrategy,
  withViewTransitions,
  withComponentInputBinding,
} from '@angular/router';
import { provideIonicAngular, IonicRouteStrategy } from '@ionic/angular/standalone';
import { provideHttpClient, withFetch, HttpClient } from '@angular/common/http';
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { addIcons } from 'ionicons';
import {
  home, compass, settings, language, colorPalette,
  notifications, star, refresh,
} from 'ionicons/icons';

import { routes } from './app.routes';
import { LanguageService } from './services/language.service';
import { ThemeService } from './services/theme.service';
import { OnboardingService } from './services/onboarding.service';
import { PurchasesService } from './services/purchases.service';
import { AdsService } from './services/ads.service';
import { AppErrorHandler } from './core/error-handler';

export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

// One-shot icon registration. Runs at app config build, before any component
// renders — replaces `addIcons(...)` calls scattered in component constructors.
addIcons({
  home, compass, settings, language, colorPalette,
  notifications, star, refresh,
});

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withViewTransitions(),         // smooth, native-feeling page transitions on supported browsers
      withComponentInputBinding(),   // route params / data bind directly to component inputs
    ),
    // `mode: 'md'` forces Material Design styling on iOS too — gives a
    // consistent look across both platforms. Drop the `mode` option to let
    // Ionic auto-detect (iOS users see iOS-native, Android users see Material).
    provideIonicAngular({ mode: 'md' }),
    provideHttpClient(withFetch()),  // withFetch() — modern default, SSR-friendly
    { provide: RouteReuseStrategy, useClass: IonicRouteStrategy },
    { provide: ErrorHandler, useClass: AppErrorHandler },
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

    // ---- Bootstrap ----
    // APP_INITIALIZER blocks the app from rendering / routing until its returned
    // Promise resolves. This is the right place for "load saved state into
    // signal stores before any guard runs" — moving it out of AppComponent.ngOnInit
    // (which runs *after* the router has already started matching the initial URL)
    // closes the cold-start race that would otherwise let the onboarding guard
    // read a default-`false` signal.
    {
      provide: APP_INITIALIZER,
      multi: true,
      useFactory: () => {
        const language = inject(LanguageService);
        const theme = inject(ThemeService);
        const onboarding = inject(OnboardingService);
        const purchases = inject(PurchasesService);
        const ads = inject(AdsService);

        return async () => {
          // Language first — LanguageService owns the resolve-saved → browser → 'en'
          // flow and awaits the translate JSON load (firstValueFrom) so first paint
          // shows real text, not raw keys.
          await language.initialize();

          // Theme + onboarding hydrate from @capacitor/preferences (parallel — no deps).
          await Promise.all([
            theme.initialize(),
            onboarding.initialize(),
          ]);

          // Purchases must complete before Ads (which reads purchases.isPremium()).
          await purchases.initialize();
          await ads.initialize();
        };
      },
    },
  ],
};
```

> **Why `APP_INITIALIZER` and not `AppComponent.ngOnInit` doing `Promise.all`?**
> `ngOnInit` runs **after** the router has already started matching the initial URL — the first guard can fire against an unhydrated signal and incorrectly redirect a returning user back to onboarding. `APP_INITIALIZER` blocks bootstrap (and therefore routing) until its Promise resolves. The signal-store guard's "the value is hydrated by the time you run" claim is then a real guarantee, not a race.

> **Note on `provideAppInitializer()`:** Angular has been moving toward a typed `provideAppInitializer(() => …)` helper that wraps `APP_INITIALIZER`. This skill uses the underlying token because it's stable across versions; if your project's Angular version exposes the helper as stable, swap the provider for `provideAppInitializer(initFactory)` — same semantics.

**`withViewTransitions()`** — wraps every route navigation in a `document.startViewTransition()` call where supported, giving you free CSS-driven page transitions. Falls back gracefully on browsers without the View Transitions API.

**`withComponentInputBinding()`** — route params and `data` bind to component inputs declaratively:

```typescript
// route definition
{ path: 'post/:id', loadComponent: () => import('./post/post.page').then(m => m.PostPage) }

// component
import { Component, input } from '@angular/core';

@Component({ /* ... */ })
export class PostPage {
  // Automatically bound from /post/:id — no inject(ActivatedRoute) needed
  readonly id = input.required<string>();
}
```

Removes the `ActivatedRoute` boilerplate from most pages.

## `core/error-handler.ts`

```typescript
import { ErrorHandler, Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class AppErrorHandler implements ErrorHandler {
  handleError(error: unknown): void {
    // In development, surface the full error to the console.
    // In production, forward to Sentry / Crashlytics — see ../../ionic-sentry/.
    console.error('[AppErrorHandler]', error);
  }
}
```

If you wire Sentry (see [`../../ionic-sentry/`](../../ionic-sentry/SKILL.md)), replace this with `Sentry.createErrorHandler({ showDialog: false })` per the Sentry skill — it satisfies the `ErrorHandler` interface and forwards to Sentry's reporting pipeline.

## `app.component.html`

```html
<ion-app>
  <ion-router-outlet></ion-router-outlet>
</ion-app>
```

## `app.component.ts`

With bootstrap moved into `APP_INITIALIZER`, `AppComponent` is a pure shell — no lifecycle hooks, no service injection.

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { IonApp, IonRouterOutlet } from '@ionic/angular/standalone';

@Component({
  selector: 'app-root',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [IonApp, IonRouterOutlet],
  templateUrl: './app.component.html',
})
export class AppComponent {}
```

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

bootstrapApplication(AppComponent, appConfig).catch(err => console.error(err));
```

`.catch(...)` — `bootstrapApplication` returns a Promise; without a catch, a bootstrap-time rejection (typically the `APP_INITIALIZER` factory throwing) becomes an unhandled rejection.
