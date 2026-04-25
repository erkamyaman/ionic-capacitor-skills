# `@defer` — Lazy-Render Heavy Chunks

`@defer` is Angular's template-level lazy loading. It splits chunks at compile time and pulls them in based on a trigger — `idle`, `viewport`, `interaction`, `hover`, `timer`, or a manual signal.

For a mobile app, the cold-start path matters. The Onboarding page pulls in Swiper (~60–100 KB once compressed), so wrapping the slide control in `@defer` is a real win. Most other pages in this scaffold are too small to benefit — `@defer` adds an HTTP roundtrip per chunk, so the savings have to clear that overhead.

## Onboarding — defer the Swiper slides

The video plays immediately (it's the background and the user must see it). The Swiper-based slide control can wait until the page is interactive:

```html
<!-- onboarding/onboarding.page.html -->
<ion-page>
  <ion-content [fullscreen]="true" class="onboarding-content">
    <video
      [src]="videoUrl"
      autoplay loop muted playsinline aria-hidden="true"
      class="background-video"
    ></video>
    <div class="gradient-overlay" aria-hidden="true"></div>

    @defer (on idle) {
      <app-onboarding-slides (complete)="completeOnboarding()" />
    } @placeholder {
      <!-- shown while the chunk hasn't loaded yet -->
      <div class="onboarding-skeleton"></div>
    } @loading (after 100ms; minimum 200ms) {
      <ion-spinner />
    }
    <!--
      `<app-onboarding-slides>` is a separate component you'll create — it
      contains the Swiper imports and slide markup. Generate it with:
          npx ng generate component onboarding/onboarding-slides --inline-style=false
      It must `output()` a `complete` event the page listens to.
    -->
  </ion-content>
</ion-page>
```

The Swiper-driven `<app-onboarding-slides>` and its `swiper` import live in their own bundle, fetched after first paint when the browser is idle.

## When NOT to defer: paywall option list

It's tempting to wrap the paywall's option-card grid in `@defer`. **Don't** — the paywall page is already lazy-loaded via `loadComponent`, and splitting the option-card markup into its own chunk just adds an extra HTTP roundtrip after `getOfferings()` resolves. The option-card markup is small enough (a few KB) that the round-trip cost exceeds the savings, and on slow networks the user sees an extra load delay.

`@defer` pays off when the deferred subtree pulls in genuinely heavy code (Swiper, a charting library, a rich-text editor). If the only thing you'd save by deferring is some `<button>` markup and a few signal reads, ship it in the page chunk. See the "Don't `@defer` everything" section below.

## Triggers

| Trigger | When the chunk fetches |
|---------|------------------------|
| `on idle` | Browser is idle (`requestIdleCallback`) — best default for non-critical UI. |
| `on viewport` | Placeholder enters the viewport. Good for below-the-fold sections. |
| `on interaction` | User taps / clicks the placeholder. Great for accordion-style UI. |
| `on hover` | Pointer hovers — desktop-only really; not useful for touch. |
| `on timer(2s)` | Fixed delay. Avoid unless you have a specific reason. |
| `on immediate` | As soon as the parent finishes rendering. Almost defeats the purpose. |
| `when expr` | When `expr` becomes truthy. Pair with signals. |

You can combine: `@defer (on viewport; on idle)` — fetches whichever fires first.

## Prefetch

Fetch eagerly (don't render yet) so when the trigger fires, render is instant:

```html
@defer (on interaction; prefetch on idle) {
  <app-heavy-modal />
} @placeholder {
  <button (click)="...">Open settings</button>
}
```

`prefetch on idle` downloads the chunk during idle time, then `on interaction` instantly renders it on tap.

## What it actually saves

The deferred component and **all its imports** become a separate chunk. Sample numbers from a real Ionic project (illustrative — yours will differ):

```
main.js                       ~150 KB   ← what loads on first paint
chunk-onboarding-slides.js    ~80 KB    ← Swiper, slide CSS, etc.
chunk-paywall-options.js      ~20 KB    ← option-card component
```

For a `swiper`-using component, that's typically 60–100 KB you don't ship in the critical path. Run `npx ng build --configuration production` and inspect the build output to see your project's actual savings.

## Don't `@defer` everything

- The route component itself is already lazy (`loadComponent`); double-deferring doesn't help.
- Tiny components (under ~5 KB chunked) aren't worth splitting — the request overhead exceeds the savings.
- Anything that must be visible on first paint (the video, the page header). Deferring those creates jank.

## Verify

```bash
npx ng build --configuration production
ls dist/<app>/browser/
# Look for chunk-*.js files corresponding to your @defer blocks
```

Or check the build output's "Initial chunk files" vs "Lazy chunk files" tables — `@defer` blocks land in the lazy section.
