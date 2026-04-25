# Filesystem (`@capacitor/filesystem`)

## Install

```bash
npm install @capacitor/filesystem
npx cap sync
```

## Directories

```typescript
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';
```

| Directory               | iOS                          | Android                       | Persisted across reinstalls? |
|-------------------------|------------------------------|-------------------------------|------------------------------|
| `Directory.Data`        | `Documents/`                 | App-internal data dir         | No |
| `Directory.Documents`   | `Documents/` (visible in Files app) | App-internal documents dir | No |
| `Directory.Library`     | `Library/`                   | Files dir                     | No |
| `Directory.Cache`       | `Library/Caches/`            | Cache dir                     | No (OS may purge) |
| `Directory.External`    | n/a                          | External SD card              | Varies |
| `Directory.ExternalStorage` | n/a                      | Android `/storage/emulated/0/` | Survives reinstall |

For most app-internal use cases, `Directory.Data`. For files the user should see in the system Files app on iOS, `Directory.Documents` (and add `UIFileSharingEnabled` to `Info.plist` to expose them).

## Read / write

```typescript
// Write a text file
await Filesystem.writeFile({
  path: 'notes/today.txt',
  data: 'Hello world',
  directory: Directory.Data,
  encoding: Encoding.UTF8,
  recursive: true,        // create parent dirs as needed
});

// Read it back
const result = await Filesystem.readFile({
  path: 'notes/today.txt',
  directory: Directory.Data,
  encoding: Encoding.UTF8,
});
console.log(result.data);

// List
const ls = await Filesystem.readdir({ path: 'notes', directory: Directory.Data });
console.log(ls.files);   // array of { name, type, size, ctime, mtime }

// Delete
await Filesystem.deleteFile({ path: 'notes/today.txt', directory: Directory.Data });
```

## Binary files

For images / PDFs / etc., omit `encoding` — data is then a base64 string:

```typescript
const photo = await Camera.getPhoto({ resultType: CameraResultType.Base64 });
await Filesystem.writeFile({
  path: 'images/avatar.jpg',
  data: photo.base64String!,
  directory: Directory.Data,
  recursive: true,
});
```

To display, use `Filesystem.getUri()` to get a `file://` URL:

```typescript
const { uri } = await Filesystem.getUri({
  path: 'images/avatar.jpg',
  directory: Directory.Data,
});
// uri is 'file:///...' — viewable via Capacitor.convertFileSrc(uri)
```

`<img>` cannot load `file://` directly in WebView. Use `Capacitor.convertFileSrc(uri)`:

```typescript
import { Capacitor } from '@capacitor/core';
const displayUrl = Capacitor.convertFileSrc(uri);
// <img :src="displayUrl" />
```

## Cleanup

App-internal directories don't auto-purge except `Cache`. If your app writes a lot, prune old files yourself.
