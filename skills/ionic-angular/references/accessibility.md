# Accessibility (ARIA) for the scaffold

Mobile apps go through store accessibility scrutiny (Apple's App Review and the Play Console both flag glaring failures) and screen-reader users are real users. The scaffold's pages have a few specific patterns to get right.

## Onboarding video — decorative

The fullscreen background video on the Onboarding page is **decorative**. It carries no information; it's vibe. Hide it from assistive tech:

```html
<!-- aria-hidden="true" makes screen readers skip it -->
<video
  [src]="videoUrl"
  autoplay
  loop
  muted
  playsinline
  aria-hidden="true"
  class="background-video"
></video>
<div class="gradient-overlay" aria-hidden="true"></div>
```

The actual onboarding content (slides, headings, buttons) sits above and gets read normally. (HTML comments are not allowed inside an element's start tag — keep them on a separate line as above.)

## Paywall option cards — single-select group

The current pattern is two `<div>`s with click handlers. To a screen reader these are unannounced regions. Either turn them into proper buttons or use `role="radio"` + `role="radiogroup"` since they're a single-select.

### Option A: `<button>` (simplest)

```html
<div class="subscription-options" role="radiogroup" [attr.aria-label]="'paywall.title' | translate">
  @for (option of subscriptionOptions; track option.id) {
    <button
      type="button"
      class="option-card"
      role="radio"
      [attr.aria-checked]="selectedPlan() === option.id"
      [class.selected]="selectedPlan() === option.id"
      (click)="selectedPlan.set(option.id)"
    >
      @if (option.badge) {
        <ion-badge color="danger">{{ option.badge }}</ion-badge>
      }
      <h3>{{ option.title | translate }}</h3>
      <p>{{ option.price }}</p>
    </button>
  }
</div>
```

`<button type="button">` gets keyboard focus, Space/Enter activation, and screen-reader announcement for free. `aria-checked` mirrors the visual selected state.

### Option B: keep `<div>` but add the roles

If the card needs to be a `<div>` for layout reasons:

```html
<div
  class="option-card"
  role="radio"
  tabindex="0"
  [attr.aria-checked]="selectedPlan() === option.id"
  (click)="selectedPlan.set(option.id)"
  (keydown.enter)="selectedPlan.set(option.id); $event.preventDefault()"
  (keydown.space)="selectedPlan.set(option.id); $event.preventDefault()"
>…</div>
```

Native `<button>` is always preferable — only do this if you can't.

## Tab bar

Ionic's `<ion-tabs>` already wires up basic ARIA, but tab buttons benefit from explicit labels (the icon alone isn't read out cleanly):

```html
<ion-tab-button tab="home" [attr.aria-label]="'tabs.home' | translate">
  <ion-icon name="home" aria-hidden="true"></ion-icon>
  <ion-label>{{ 'tabs.home' | translate }}</ion-label>
</ion-tab-button>
```

`aria-hidden="true"` on the icon prevents the screen reader from announcing the icon name on top of the label.

## Settings selects

`<ion-select>` is accessible by default — Ionic gives it the right role and announces the open/close state. Make sure each item has a clear `<ion-label>`:

```html
<ion-item>
  <ion-icon name="language" slot="start" aria-hidden="true"></ion-icon>
  <ion-label>{{ 'settings.language' | translate }}</ion-label>
  <ion-select
    [value]="currentLang()"
    [attr.aria-label]="'settings.language' | translate"
    (ionChange)="changeLanguage($event)"
  >
    <ion-select-option value="en">English</ion-select-option>
    <ion-select-option value="tr">Türkçe</ion-select-option>
  </ion-select>
</ion-item>
```

The `aria-label` on `<ion-select>` itself ensures the select control announces its purpose even when the visual `<ion-label>` is to the side.

## Modal / sheet titles

For any custom modal or sheet (paywall is sometimes presented modally), set an `aria-labelledby` pointing to the title:

```html
<ion-content aria-labelledby="paywall-title">
  <h1 id="paywall-title">{{ 'paywall.title' | translate }}</h1>
  …
</ion-content>
```

## Touch targets

Apple HIG specifies **44×44 pt** minimum (iOS); Material 3 specifies **48×48 dp** minimum (Android). On a high-density Android display, `44px` resolves to roughly 30 dp — below the Material guidance. Use `48px` so the same SCSS satisfies both platforms.

```scss
.option-card {
  min-height: 48px;     // 48 dp on Android (Material), >= 44 pt on iOS
  min-width: 48px;
  padding: 16px;
}
```

`<ion-button>` defaults already satisfy both; this only matters for custom card buttons / pressable `<div>`-equivalents.

## Color contrast

The default Ionic palettes pass WCAG AA. If you customize CSS variables (e.g., `--ion-color-primary`), verify the new color against contrast tools — Chrome DevTools' Lighthouse audit catches most issues.

## Dynamic announcements

For status changes the user should hear (purchase succeeded, settings saved), use a polite live region:

```html
<div role="status" aria-live="polite" class="visually-hidden">
  {{ statusMessage() }}
</div>
```

```scss
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

Update `statusMessage()` after async actions; the screen reader reads it without interrupting.

## Quick audit

```bash
# Browser-based — Lighthouse a11y audit on `ionic serve`
# Native — iOS: Settings → Accessibility → Accessibility Inspector (Xcode)
#          Android: Accessibility Scanner app from Play Store
```

## Don't

- Don't put click handlers on `<div>` / `<span>` / `<ion-icon>` without keyboard support and a `role`.
- Don't use color alone to communicate state (the "selected" card needs a checkmark or border, not just a background tint).
- Don't auto-play sound. Set `muted` on every video. Apple already requires this for autoplay — same for screen-reader sanity.
- Don't trap focus in modals you didn't write yourself. Ionic handles this; custom modals must trap focus and restore it on dismiss.
