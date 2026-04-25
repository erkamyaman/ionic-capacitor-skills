# Supabase Setup

## Console

1. <https://supabase.com/dashboard> → **New project**.
2. Pick region close to your users.
3. From **Project Settings → API**, copy:
   - **Project URL** (`https://xxxx.supabase.co`)
   - **anon public** key (publishable)
4. **Don't copy** the service role key into the app — that one stays server-side.

## Env vars

Following [`../../ionic-shared/references/environments-and-keys.md`](../../ionic-shared/references/environments-and-keys.md):

```env
# .env.development
VITE_SUPABASE_URL=https://xxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOi...
```

## Client init with Capacitor Preferences storage

The default JS client uses `localStorage` for the auth session. On mobile, `@capacitor/preferences` is more reliable (survives WebView storage clears, encrypted on iOS):

```typescript
// utils/supabase.ts
import { createClient, SupportedStorage } from '@supabase/supabase-js';
import { Preferences } from '@capacitor/preferences';

const capacitorStorage: SupportedStorage = {
  async getItem(key) {
    const { value } = await Preferences.get({ key });
    return value;
  },
  async setItem(key, value) {
    await Preferences.set({ key, value });
  },
  async removeItem(key) {
    await Preferences.remove({ key });
  },
};

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY,
  {
    auth: {
      storage: capacitorStorage,
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: false,   // we handle deep links manually
    },
  },
);
```

Typing `capacitorStorage` as `SupportedStorage` catches mismatches at compile time — `as never` / `as unknown` would silence real type errors if Supabase ever changes the interface.

`detectSessionInUrl: false` is important — Supabase's default tries to parse the OAuth callback from `window.location`, which doesn't work the same in a native WebView. Handle the callback via `@capacitor/app`'s `appUrlOpen` instead (see `auth.md`).

## Row Level Security (RLS) — required before launch

Open the SQL editor and for every user-data table:

```sql
alter table public.posts enable row level security;

-- Allow logged-in users to read all posts
create policy "Anyone can read posts"
  on public.posts for select
  using (auth.role() = 'authenticated');

-- Allow users to insert their own posts
create policy "Users can insert their own posts"
  on public.posts for insert
  with check (auth.uid() = author_id);

-- Update / delete only own
create policy "Users can update their own posts"
  on public.posts for update
  using (auth.uid() = author_id);

create policy "Users can delete their own posts"
  on public.posts for delete
  using (auth.uid() = author_id);
```

Without RLS, anyone with the anon key (every user of your app, plus anyone who decompiles the bundle) can read/write everything. With RLS enabled but no policies, all queries return empty — also problematic. Enable + add policies together.

## Local dev

```bash
npx supabase init
npx supabase start
```

Spins up Postgres + Auth + Studio locally. Point the app at `http://localhost:54321` in dev.

## Migrations

```bash
npx supabase migration new add_posts_table
# edits supabase/migrations/<timestamp>_add_posts_table.sql
npx supabase db reset    # apply locally
npx supabase db push     # push to remote
```

Treat the `supabase/` folder as committed schema — review migrations in PRs the same as code.
