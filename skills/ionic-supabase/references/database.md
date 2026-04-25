# Supabase Database (Postgres)

PostgREST-style queries on top of Postgres. Realtime subscriptions are opt-in per table.

## Read

```typescript
import { supabase } from '../utils/supabase';

// Select all columns
const { data, error } = await supabase
  .from('posts')
  .select('*');

// Specific columns + filter + order + limit
const { data, error } = await supabase
  .from('posts')
  .select('id, title, created_at')
  .eq('author_id', userId)
  .order('created_at', { ascending: false })
  .limit(20);

// Joins via foreign key
const { data, error } = await supabase
  .from('posts')
  .select(`
    id, title,
    author:users (id, display_name, avatar_url)
  `);
```

The Supabase client returns `{ data, error }` — always check `error` before using `data`.

## Insert / update / upsert / delete

```typescript
// Insert one
const { data } = await supabase
  .from('posts')
  .insert({ title: 'Hello', body: '...' })
  .select()
  .single();

// Insert many
await supabase.from('posts').insert([
  { title: 'A', body: '...' },
  { title: 'B', body: '...' },
]);

// Update
await supabase
  .from('posts')
  .update({ title: 'Edited' })
  .eq('id', postId);

// Upsert (insert or update on PK conflict)
await supabase
  .from('posts')
  .upsert({ id: postId, title: 'X', body: 'Y' });

// Delete
await supabase
  .from('posts')
  .delete()
  .eq('id', postId);
```

## RLS reminder

Every query above runs through Postgres RLS using the user's JWT. If RLS is enabled but no policy allows the operation, you'll get an empty result (for selects) or a permission error (for writes).

The classic mistake: enable RLS, forget to add policies, then wonder why all selects return `[]`.

## Realtime subscription

```typescript
import { supabase } from '../utils/supabase';

const channel = supabase
  .channel('posts-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'posts' },
    (payload) => {
      // payload.eventType: 'INSERT' | 'UPDATE' | 'DELETE'
      // payload.new, payload.old
    },
  )
  .subscribe();

// Cleanup
await supabase.removeChannel(channel);
```

Realtime requires enabling replication on the table in the Supabase dashboard → Database → Publications.

For **filtered** realtime (e.g. only this user's posts):

```typescript
.on(
  'postgres_changes',
  { event: '*', schema: 'public', table: 'posts', filter: `author_id=eq.${userId}` },
  handler,
)
```

## Storage

For files (avatars, attachments):

```typescript
// Upload
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.jpg`, fileBlob, {
    upsert: true,
    contentType: 'image/jpeg',
  });

// Public URL
const { data: { publicUrl } } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/avatar.jpg`);

// Signed URL (private buckets)
const { data: { signedUrl } } = await supabase.storage
  .from('private-files')
  .createSignedUrl(`${userId}/secret.pdf`, 60 * 60); // 1 hour
```

Bucket policies are configured in the dashboard — they're separate from table RLS but work the same conceptually.

## RPC (Postgres functions)

Encapsulate complex logic in Postgres functions and call them from the client:

```sql
create or replace function get_user_stats(uid uuid)
returns table (post_count int, comment_count int)
language sql
security invoker  -- runs as the calling user; respects RLS
as $$
  select
    (select count(*) from posts where author_id = uid)::int,
    (select count(*) from comments where author_id = uid)::int;
$$;
```

```typescript
const { data } = await supabase.rpc('get_user_stats', { uid: userId });
```

`security invoker` is the safe default — `security definer` runs as the function owner (typically `postgres`), bypassing RLS, which is occasionally what you want but easy to misuse.
