# Share (`@capacitor/share`)

Native iOS share sheet / Android share intent.

## Install

```bash
npm install @capacitor/share
npx cap sync
```

## Basic share

```typescript
import { Share } from '@capacitor/share';

await Share.share({
  title: 'Check this out',
  text: 'I found this great article',
  url: 'https://example.com/article/123',
  dialogTitle: 'Share with friends',  // Android only — title above the sheet
});
```

All four fields are optional but at least `text` or `url` should be present.

## Capability check

Returns `false` on web (Web Share API is not always available) and platforms that don't support it:

```typescript
const { value } = await Share.canShare();
if (!value) {
  // Show fallback UI: copy-to-clipboard, etc.
}
```

## Share files

Capacitor's `Share` plugin doesn't directly accept files — for sharing files, write the file via Filesystem first, then share the resulting `file://` URI:

```typescript
const { uri } = await Filesystem.getUri({
  path: 'export.pdf',
  directory: Directory.Cache,
});
await Share.share({
  title: 'Your export',
  url: uri,                  // file:// URI — iOS and Android can share files
  dialogTitle: 'Share PDF',
});
```

For sharing a generated image (e.g., a screenshot), the same pattern applies — write the image to `Directory.Cache`, then share its URI.

## What gets shared on each platform

- **iOS**: shows the system share sheet with all installed apps that handle the content type. URLs render as link previews.
- **Android**: shows the share intent picker. Apps that registered an `intent-filter` for the content type appear.
- **Web**: uses `navigator.share()` if available (Chrome, Safari mobile); falls back via `canShare() === false`.

## Common mistake

Don't try to share a `data:` URL — it's not a file. Write to disk first.
