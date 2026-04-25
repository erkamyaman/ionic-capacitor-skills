---
name: ionic-supabase
description: Wire Supabase Auth and Postgres database into an Ionic Capacitor app via the @supabase/supabase-js client. Trigger when adding Supabase Auth (email, magic link, Google, Apple), Postgres queries, realtime subscriptions, or storage to an Ionic app.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Supabase

Open-source Firebase alternative: Postgres + GoTrue auth + Realtime + Storage. Same shape as `ionic-firebase` but Postgres-flavored — pick this when your team prefers SQL, RLS, or self-hosting.

## When to consult

- **Project setup + client init**: [setup.md](references/setup.md)
- **Auth** (email, magic link, OAuth, native Apple/Google): [auth.md](references/auth.md)
- **Database** (queries, RLS, realtime): [database.md](references/database.md)
- **Per-framework integration**: [framework-snippets.md](references/framework-snippets.md)

## Hard rules

- ✅ The **anon key** is publishable — safe to ship in the bundle. Row Level Security (RLS) does the gating.
- ✅ The **service role key** is a secret — NEVER ship it; server-side only.
- ✅ **Enable RLS on every table** that holds user data. Without RLS, anon-key clients can read/write everything.
- ✅ Native OAuth (Google / Apple) on mobile uses `signInWithOAuth({ skipBrowserRedirect: true })` + your app's deep-link scheme. The default web flow doesn't return cleanly to a native app.
- ❌ Don't store JWTs in `localStorage` if you can avoid it — use `@capacitor/preferences` via the Supabase storage adapter (see setup.md).

## Library

```bash
npm install @supabase/supabase-js
```

No native plugin required — the JS client works in WKWebView / Android WebView and uses HTTPS only.
