---
name: ionic-native-essentials
description: Six small Capacitor plugins bundled — Camera, Filesystem, Share, Haptics, Network, Keyboard. Trigger when adding photo upload, file save/read, native share sheet, haptic feedback, network status detection, or keyboard handling to an Ionic Capacitor app.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Native Essentials

Six small but commonly needed Capacitor plugins. Each gets one focused reference; install only what you actually use.

## When to consult

- **Camera** (`@capacitor/camera`) — take photos / pick from gallery: [camera.md](references/camera.md)
- **Filesystem** (`@capacitor/filesystem`) — read / write files in app sandbox: [filesystem.md](references/filesystem.md)
- **Share** (`@capacitor/share`) — native share sheet: [share.md](references/share.md)
- **Haptics** (`@capacitor/haptics`) — taps, vibrations, impact feedback: [haptics.md](references/haptics.md)
- **Network** (`@capacitor/network`) — connection status + offline UX: [network.md](references/network.md)
- **Keyboard** (`@capacitor/keyboard`) — show/hide / resize behavior on focus: [keyboard.md](references/keyboard.md)

## Hard rules

- ✅ Install only the plugins you use — don't install all six "just in case." Each adds binary size.
- ✅ Run `npx cap sync` after every plugin install.
- ✅ Add the relevant `Info.plist` permission strings for plugins that need them (Camera = `NSCameraUsageDescription`, Filesystem with photo library access = `NSPhotoLibraryUsageDescription`).
- ❌ Don't request permissions on launch — request when the user invokes the feature (tap "Take photo", not on app start).
