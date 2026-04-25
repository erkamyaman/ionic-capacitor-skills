# Commands (Angular)

> Use `npx ng <cmd>` as the canonical CLI. `ionic serve` / `ionic build` are wrappers from the Ionic CLI — they work fine but are an extra dependency. `ng serve` and `ng build` give you the same result and use the Angular CLI directly.

## After completing code (always run)

```bash
npm install        # if dependencies changed
npx ng build       # verify the project compiles
npx cap sync       # copy web assets + plugins to native projects
```

If the build fails, fix the errors before reporting the task as done. Do not skip these steps.

## Development

```bash
npx ng serve             # browser dev server (or: ionic serve)
npx ng build             # production build  (or: ionic build)
npx cap sync             # sync to native after each build
npx cap open ios         # open Xcode
npx cap open android     # open Android Studio
npx cap run ios          # build + run on iOS
npx cap run android      # build + run on Android
```

## Generating new code

```bash
npx ng generate component <name>
npx ng generate service <name>
npx ng generate guard <name>
npx ng generate pipe <name>
npx ng generate directive <name>
```

Take note of the path printed by `ng generate` so subsequent edits target the right files. Always augment the generated code — don't rewrite from scratch — to keep CLI conventions intact.

## Pre-release

```bash
npm run build
npx cap sync
npx cap open ios       # archive & distribute
npx cap open android   # signed bundle
```

See [`../../ionic-shared/references/app-store-notes.md`](../../ionic-shared/references/app-store-notes.md) for the full submission checklist.
