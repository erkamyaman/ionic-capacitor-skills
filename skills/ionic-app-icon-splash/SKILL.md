---
name: ionic-app-icon-splash
description: Generate every iOS and Android app icon and splash screen size from a single source PNG using @capacitor/assets. Trigger when adding app icons, generating splash screens, replacing the default Capacitor icon, or shipping any Ionic Capacitor app to a store (icons are non-optional).
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# App Icon & Splash Generation

iOS and Android each need ~25 icon sizes and 4+ splash variants. `@capacitor/assets` generates all of them from one source PNG (icon) and one source PNG (splash) and copies them into the right native folders. Replace the default Capacitor icon before any store submission.

## When to consult

- **Source asset specs + generation command**: [generate.md](references/generate.md)
- **iOS / Android specifics + light/dark splashes**: [platform-specifics.md](references/platform-specifics.md)

## Hard rules

- ✅ Source icon: **1024×1024 PNG**, no transparency, no rounded corners (iOS adds the mask).
- ✅ Source splash: **2732×2732 PNG**, centered logo, solid background.
- ✅ Re-run `npx capacitor-assets generate` whenever the source PNG changes.
- ✅ Run `npx cap sync` after generation so the new files end up in the native projects.
- ❌ Don't hand-edit the generated files in `ios/App/App/Assets.xcassets/AppIcon.appiconset/` or `android/app/src/main/res/mipmap-*/` — the next generation overwrites them.
- ❌ Don't ship the default Capacitor compass icon. App Review notices.

## Library

```bash
npm install --save-dev @capacitor/assets
```
