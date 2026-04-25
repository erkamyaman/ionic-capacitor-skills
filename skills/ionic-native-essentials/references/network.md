# Network (`@capacitor/network`)

Detect connection status; react to changes for offline UX.

## Install

```bash
npm install @capacitor/network
npx cap sync
```

## API

```typescript
import { Network } from '@capacitor/network';

// One-shot
const status = await Network.getStatus();
console.log(status);
// { connected: true, connectionType: 'wifi' | 'cellular' | 'none' | 'unknown' }

// Subscribe to changes
const handle = await Network.addListener('networkStatusChange', (s) => {
  console.log('Network changed:', s.connected, s.connectionType);
});

// Cleanup
handle.remove();
```

## Offline-aware UX patterns

### Banner when offline

Maintain a global "online" signal/state and render a top banner when disconnected:

```typescript
// utils/network-state.ts
import { Network } from '@capacitor/network';

let listeners: Array<(online: boolean) => void> = [];
let online = true;

export async function startNetworkWatch() {
  const status = await Network.getStatus();
  online = status.connected;
  await Network.addListener('networkStatusChange', (s) => {
    online = s.connected;
    listeners.forEach(fn => fn(online));
  });
}

export function isOnline() { return online; }
export function onNetworkChange(fn: (online: boolean) => void) {
  listeners.push(fn);
  return () => { listeners = listeners.filter(l => l !== fn); };
}
```

Wrap in a service / hook / composable per framework, then expose to the layout component to render a banner.

### Queue + retry when offline

For mutations (POST / PATCH), queue them in `@capacitor/preferences` while offline, then flush when online:

```typescript
async function send(req: Request) {
  if (!isOnline()) {
    const queue = JSON.parse((await Preferences.get({ key: 'pendingReqs' })).value ?? '[]');
    queue.push(serialize(req));
    await Preferences.set({ key: 'pendingReqs', value: JSON.stringify(queue) });
    return;
  }
  return fetch(req);
}

onNetworkChange(async (online) => {
  if (!online) return;
  const queue = JSON.parse((await Preferences.get({ key: 'pendingReqs' })).value ?? '[]');
  for (const item of queue) await fetch(deserialize(item));
  await Preferences.remove({ key: 'pendingReqs' });
});
```

For richer offline-first behavior (cache reads, conflict resolution), look at libraries like [TanStack Query](https://tanstack.com/query) — they have built-in offline support patterns.

## Don't trust `connectionType: 'wifi'`

It just means the device is on Wi-Fi — that Wi-Fi could still be a captive portal. The only reliable check is "did my actual API request succeed?"
