# Commands (Vue)

## After completing code (always run)

```bash
npm install        # if dependencies changed
npm run build      # verify the project compiles
npx cap sync       # copy web assets + plugins to native projects
```

If the build fails, fix the errors before reporting the task as done. Do not skip these steps.

## Development

```bash
ionic serve              # browser dev server
ionic build              # production build
npx cap sync             # sync to native after each build
npx cap open ios         # open Xcode
npx cap open android     # open Android Studio
npx cap run ios          # build + run on iOS
npx cap run android      # build + run on Android
```

## Pre-release

```bash
npm run build
npx cap sync
npx cap open ios       # archive & distribute
npx cap open android   # signed bundle
```

See [`../../ionic-shared/references/app-store-notes.md`](../../ionic-shared/references/app-store-notes.md) for the full submission checklist.
