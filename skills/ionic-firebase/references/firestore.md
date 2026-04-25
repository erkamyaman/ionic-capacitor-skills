# Cloud Firestore

Document-oriented NoSQL with realtime listeners and offline persistence.

## Document write

```typescript
import { doc, setDoc, addDoc, collection, serverTimestamp } from 'firebase/firestore';
import { db } from './firebase';

// Set a known doc (overwrites)
await setDoc(doc(db, 'users', userId), {
  displayName: 'Alice',
  createdAt: serverTimestamp(),
});

// Set with merge (partial update without overwrite)
await setDoc(doc(db, 'users', userId), { lastSeenAt: serverTimestamp() }, { merge: true });

// Add to collection (auto-generated ID)
const ref = await addDoc(collection(db, 'posts'), {
  authorId: userId,
  title: 'Hello',
  body: '...',
  createdAt: serverTimestamp(),
});
console.log(ref.id);
```

## Document read

```typescript
import { doc, getDoc } from 'firebase/firestore';

const snap = await getDoc(doc(db, 'users', userId));
if (snap.exists()) console.log(snap.data());
```

## Query a collection

```typescript
import { collection, query, where, orderBy, limit, getDocs } from 'firebase/firestore';

const q = query(
  collection(db, 'posts'),
  where('authorId', '==', userId),
  orderBy('createdAt', 'desc'),
  limit(20),
);

const snap = await getDocs(q);
snap.forEach(d => console.log(d.id, d.data()));
```

Compound queries (e.g., `where + orderBy` on different fields) require a **composite index** — Firestore prints a console URL the first time you run an unindexed query; click it to one-click create.

## Realtime listener

```typescript
import { onSnapshot, query, collection, orderBy } from 'firebase/firestore';

const unsubscribe = onSnapshot(
  query(collection(db, 'posts'), orderBy('createdAt', 'desc')),
  (snap) => {
    const posts = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    // Update component state
  },
);

// On unmount
unsubscribe();
```

In React/Vue, set up the subscription in a `useEffect` / `onMounted` and call `unsubscribe()` in cleanup. In Angular, do the same in a service / signal — or use `@angular/fire` for typed observables.

## Offline persistence

The Firebase JS SDK's default cache is **memory-only** — `getFirestore(app)` (as shown in [setup.md](setup.md)) gives you in-memory caching that's lost on every app launch. To enable IndexedDB-backed offline persistence, opt in at init:

```typescript
import {
  initializeFirestore,
  persistentLocalCache,
  persistentSingleTabManager,
} from 'firebase/firestore';

const db = initializeFirestore(app, {
  localCache: persistentLocalCache({
    tabManager: persistentSingleTabManager({}),
  }),
});
```

> **Init ordering matters.** `initializeFirestore(...)` MUST be called **before any other Firestore call** for the given `app`. The setup snippet in [setup.md](setup.md) uses `getFirestore(app)` — that's the simple path with default settings. If you switch to `initializeFirestore` for offline-cache control, **replace** the `getFirestore(app)` call in `utils/firebase.ts` with `initializeFirestore(...)`. Calling `initializeFirestore` after `getFirestore` (or after any read/write) throws `Firestore has already been initialized`.

For **native** offline persistence (much faster than IndexedDB in WebView), use `@capacitor-firebase/firestore` instead — it routes reads/writes through the native iOS / Android Firestore SDKs.

## Batched writes / transactions

```typescript
import { writeBatch, doc, runTransaction, increment } from 'firebase/firestore';

// Batch — atomic across multiple docs
const batch = writeBatch(db);
batch.set(doc(db, 'users', userId), { displayName: 'Alice' });
batch.update(doc(db, 'stats', 'global'), { userCount: increment(1) });
await batch.commit();

// Transaction — read + write atomically (with retry)
await runTransaction(db, async (tx) => {
  const ref = doc(db, 'counters', 'visits');
  const snap = await tx.get(ref);
  const next = (snap.data()?.count ?? 0) + 1;
  tx.set(ref, { count: next });
});
```

## Cost / performance

- Each document read counts as a billable read — even on a realtime listener every snapshot can re-fetch (use `where` filters tightly).
- `onSnapshot` keeps a websocket open — fine for 1–5 listeners, bad if you have 50+.
- Pagination via `startAfter(lastDoc)`. Don't use `offset`-style pagination — Firestore doesn't have it efficiently.

## Don't store anything sensitive client-side

Anything readable in Firestore by the client is readable by the user (use Security Rules to enforce). For server-side secrets / admin operations, use Cloud Functions.
