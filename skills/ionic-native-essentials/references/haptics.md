# Haptics (`@capacitor/haptics`)

Tactile feedback. Subtle, but adds a feel of polish — successful purchase, error toast, button press.

## Install

```bash
npm install @capacitor/haptics
npx cap sync
```

## API

```typescript
import { Haptics, ImpactStyle, NotificationType } from '@capacitor/haptics';

// Three impact intensities — for taps, button presses
await Haptics.impact({ style: ImpactStyle.Light });
await Haptics.impact({ style: ImpactStyle.Medium });
await Haptics.impact({ style: ImpactStyle.Heavy });

// Three notification patterns — for outcomes
await Haptics.notification({ type: NotificationType.Success });
await Haptics.notification({ type: NotificationType.Warning });
await Haptics.notification({ type: NotificationType.Error });

// Selection feedback — when a value changes during a swipe / scroll
await Haptics.selectionStart();
await Haptics.selectionChanged();
await Haptics.selectionEnd();

// Generic vibration (Android primarily; iOS approximates)
await Haptics.vibrate({ duration: 300 });
```

## When to use what

| Pattern | Use for |
|---------|---------|
| `impact: Light` | Toggle on/off, switching tabs, soft button press |
| `impact: Medium` | Primary button press, drag-and-drop snap |
| `impact: Heavy` | "Big" actions — purchase confirm, major commit |
| `notification: Success` | Purchase complete, save succeeded |
| `notification: Warning` | Form validation issue, mild caution |
| `notification: Error` | Login failed, network error toast |
| `selection*` | Picker scroll, slider tick |

## Don't overdo it

Haptics on every tap becomes annoying. Reserve them for:

- Outcomes (success/error after async actions).
- Large state transitions (tab switch, modal dismiss).
- Continuous controls (slider drag).

## No iOS permission needed

Haptics work without permission. On Android, the `VIBRATE` permission is added automatically by the plugin.

## Doesn't work on simulator

iOS Simulator and Android emulator don't render haptic feedback. Test on a physical device.
