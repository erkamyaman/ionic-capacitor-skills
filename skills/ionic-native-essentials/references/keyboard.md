# Keyboard (`@capacitor/keyboard`)

Hide / show, listen for keyboard events, control resize behavior.

## Install

```bash
npm install @capacitor/keyboard
npx cap sync
```

## API

```typescript
import { Keyboard, KeyboardResize } from '@capacitor/keyboard';

await Keyboard.show();
await Keyboard.hide();

// Listeners
Keyboard.addListener('keyboardWillShow', (info) => {
  console.log('keyboard height:', info.keyboardHeight);
});
Keyboard.addListener('keyboardDidShow', (info) => { /* ... */ });
Keyboard.addListener('keyboardWillHide', () => { /* ... */ });
Keyboard.addListener('keyboardDidHide', () => { /* ... */ });

// Configure resize behavior — iOS only.
// (On Android, resize is controlled by `windowSoftInputMode` in AndroidManifest.xml — see below.)
await Keyboard.setResizeMode({ mode: KeyboardResize.Body });   // default
await Keyboard.setResizeMode({ mode: KeyboardResize.Ionic });  // recommended for Ionic apps
await Keyboard.setResizeMode({ mode: KeyboardResize.Native }); // let native handle
await Keyboard.setResizeMode({ mode: KeyboardResize.None });   // no resize

// Hide accessory bar (iOS — the bar above the keyboard with Done / arrows)
await Keyboard.setAccessoryBarVisible({ isVisible: false });

// Disable WebView scroll-on-keyboard behavior — iOS only.
// Pair with KeyboardResize.None for full effect.
await Keyboard.setScroll({ isDisabled: true });
```

## Recommended `capacitor.config.ts`

```typescript
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.company.appname',
  appName: 'App Name',
  webDir: 'dist',
  plugins: {
    Keyboard: {
      resize: 'ionic',          // matches @ionic-team/ionic page transitions
      resizeOnFullScreen: true,
    },
  },
};

export default config;
```

`resize: 'ionic'` plays best with Ionic page lifecycle and avoids the layout jitter you get with the default mode on `<ion-content>`-heavy pages.

## Common patterns

### Dismiss on outside tap

```typescript
import { IonContent } from '@ionic/...';

// On any tap outside an input, hide
const handleContentClick = () => Keyboard.hide();
```

### Adjust footer position

If you have a fixed footer (e.g., a chat input), the `keyboardWillShow` event gives the height — translate the footer up by that much:

```typescript
Keyboard.addListener('keyboardWillShow', (info) => {
  footerEl.style.transform = `translateY(-${info.keyboardHeight}px)`;
});
Keyboard.addListener('keyboardWillHide', () => {
  footerEl.style.transform = '';
});
```

`willShow` fires before the animation; `didShow` after. For animations that match the keyboard's, use `willShow` and a CSS transition with the same duration.

## Android note

Android's resize behavior is configured via `windowSoftInputMode` in `AndroidManifest.xml` (`<activity android:windowSoftInputMode="adjustResize">`) — this is set by Capacitor's default template.
