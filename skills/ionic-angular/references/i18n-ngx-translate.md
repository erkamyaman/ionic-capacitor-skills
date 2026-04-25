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

## Detect browser language at startup

```typescript
// app.component.ts
import { Component, OnInit } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';

@Component({ /* ... */ })
export class AppComponent implements OnInit {
  constructor(private translate: TranslateService) {}

  ngOnInit() {
    const browserLang = navigator.language.split('-')[0];
    this.translate.setDefaultLang('en');
    this.translate.use(['en', 'tr'].includes(browserLang) ? browserLang : 'en');
  }
}
```

## Usage in TypeScript

```typescript
import { TranslateService } from '@ngx-translate/core';

constructor(private translate: TranslateService) {}

const text = this.translate.instant('paywall.subscribe');
this.translate.get('paywall.subscribe').subscribe(text => /* ... */);
```

`instant()` is synchronous and returns the string immediately if translations are loaded; `get()` is async (returns an Observable) and is the right call when translations might not yet be ready.
