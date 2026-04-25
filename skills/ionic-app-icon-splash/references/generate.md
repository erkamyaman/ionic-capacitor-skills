# Generate Assets

## Source files

Create an `assets/` folder at the project root (sibling of `ios/`, `android/`, `src/`) with:

```
assets/
├── icon-only.png           # 1024×1024, no padding, no transparency
├── icon-foreground.png     # 1024×1024, foreground layer for adaptive icons (Android 8+)
├── icon-background.png     # 1024×1024, solid background or gradient for adaptive icons
├── splash.png              # 2732×2732, centered logo, light-mode background
└── splash-dark.png         # 2732×2732, centered logo, dark-mode background (optional)
```

Minimum viable set: just `icon-only.png` and `splash.png`. The adaptive-icon and dark-splash files are recommended but not required.

## Specs

| File | Size | Notes |
|------|------|-------|
| `icon-only.png` | 1024×1024 | iOS and Android fallback. Solid or PNG-flat background; iOS will mask to its rounded square / squircle shape automatically. |
| `icon-foreground.png` | 1024×1024 | Android adaptive icon foreground layer. The launcher applies its own mask, so keep critical content within the centered ~66% safe zone. |
| `icon-background.png` | 1024×1024 | Android adaptive icon background layer. Solid color or simple gradient. |
| `splash.png` | 2732×2732 | Centered logo within ~30% of canvas; rest is solid background. Same image scaled for every device. |
| `splash-dark.png` | 2732×2732 | Optional dark-mode splash (iOS / Android system theme). |

## Generate

```bash
npx capacitor-assets generate
```

This reads `assets/`, generates the platform-specific sizes, and writes them into:

- `ios/App/App/Assets.xcassets/AppIcon.appiconset/`
- `ios/App/App/Assets.xcassets/Splash.imageset/`
- `android/app/src/main/res/mipmap-*/`
- `android/app/src/main/res/drawable*/`

Then sync:

```bash
npx cap sync
```

## Re-generation

Anytime the source PNGs change, re-run both commands. `capacitor-assets` overwrites the previous output cleanly.

## CI

If you build via CI, add the generation step to the pipeline:

```yaml
- run: npx capacitor-assets generate
- run: npx cap sync
- run: npm run build
```

That way designers can drop new artwork into `assets/` and CI regenerates without manual intervention.
