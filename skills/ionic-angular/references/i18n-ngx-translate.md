# i18n with `@ngx-translate/core`

Translation JSON is loaded over HTTP from `assets/i18n/`. Setup lives in `app.config.ts` — see [app-config.md](app-config.md). Translation content lives in [`../../ionic-shared/references/localization-content.md`](../../ionic-shared/references/localization-content.md).

## Usage in templates

```html
{{ 'settings.title' | translate }}
```

For attributes:

```html
<ion-input [placeholder]="'paywall.title' | translate"></ion-input>
```

## Why ngx-translate vs `@angular/localize`

This skill uses [`@ngx-translate/core`](https://github.com/ngx-translate/core) — runtime translation loaded as JSON. Angular ships its own [`@angular/localize`](https://angular.dev/guide/i18n) for build-time message extraction with per-locale bundles. The trade-off:

| | `@ngx-translate/core` | `@angular/localize` |
|---|---|---|
| Switching locale at runtime | ✅ free | ❌ requires reloading per-locale bundle |
| Bundle shape | one bundle + JSON fetched per locale | one bundle per locale (build N times) |
| Maintainer | community | Angular team |
| ICU messages / pluralization | basic | first-class |
| Best for | small-locale apps that switch frequently | larger / static-locale apps, vendor-translated XLIFF |

We chose ngx-translate for runtime convenience (mobile users sometimes switch language without reinstalling). For an app that ships to broad markets with vendor-translated content, prefer `@angular/localize`.

## Detect browser language at startup

Language is owned by `LanguageService` ([services.md](services.md)). The service runs in the `APP_INITIALIZER` ([app-config.md](app-config.md)) so it's set before any guard / page renders, and it `await firstValueFrom(translate.use(lang))` so the JSON is loaded before first paint — without that wait, the first paint shows raw translation keys (`paywall.title`) until the HTTP fetch resolves.

The bootstrap path:

```
APP_INITIALIZER
  ↓
language.initialize()        ← reads Preferences → browser → 'en'; awaits JSON load
  ↓
theme + onboarding init      ← Promise.all
  ↓
purchases.initialize()       ← then ads
```

Pages call `inject(LanguageService).setLang('tr')` to change at runtime — the service handles persistence and the JSON-load wait.

## Usage in TypeScript

```typescript
import { Component, inject } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';

@Component({ /* ... */ })
export class SomePage {
  private translate = inject(TranslateService);

  showSubscribeText() {
    const text = this.translate.instant('paywall.subscribe');
    this.translate.get('paywall.subscribe').subscribe(text => /* ... */);
  }
}
```

`instant()` is synchronous and returns the string immediately if translations are loaded; `get()` is async (returns an Observable) and is the right call when translations might not yet be ready.
