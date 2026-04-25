# Camera (`@capacitor/camera`)

## Install

```bash
npm install @capacitor/camera
npx cap sync
```

## `Info.plist` (iOS)

```xml
<key>NSCameraUsageDescription</key>
<string>Take photos to attach to your posts.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Choose existing photos to attach to your posts.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Save photos you create in the app to your library.</string>
```

Apple rejects builds that lack human-readable descriptions when the camera API is referenced.

## Android permissions

`@capacitor/camera` requests runtime permissions automatically — no manual `AndroidManifest.xml` changes needed in modern Capacitor.

## Usage

```typescript
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';

// Take a photo
const photo = await Camera.getPhoto({
  quality: 90,
  allowEditing: false,
  resultType: CameraResultType.Uri,    // or DataUrl, Base64
  source: CameraSource.Camera,         // or Photos, Prompt
});

console.log(photo.webPath);            // file:// URL viewable in <img src="...">
```

`resultType` choices:

| Type        | Returns | Use when |
|-------------|---------|----------|
| `Uri`       | `webPath` (`file://...`) | Display in `<img>`, upload via `fetch(blob)` |
| `DataUrl`   | `dataUrl` (`data:image/jpeg;base64,...`) | Quick display, small images |
| `Base64`    | `base64String` | Sending to APIs that want raw base64 |

`Uri` is the right default — DataUrl/Base64 inflate memory by ~33% and stutter on large photos.

## Upload pattern

```typescript
const photo = await Camera.getPhoto({
  resultType: CameraResultType.Uri,
  source: CameraSource.Photos,
});

const blob = await fetch(photo.webPath!).then(r => r.blob());
const formData = new FormData();
formData.append('image', blob, 'upload.jpg');

await fetch('/api/upload', { method: 'POST', body: formData });
```

## Permissions API

```typescript
const status = await Camera.checkPermissions();
if (status.camera === 'prompt' || status.camera === 'prompt-with-rationale') {
  await Camera.requestPermissions({ permissions: ['camera'] });
}
```

`getPhoto()` requests permission automatically on first call — only `checkPermissions` first if you need to gate the UI (e.g., hide the camera button if denied).
