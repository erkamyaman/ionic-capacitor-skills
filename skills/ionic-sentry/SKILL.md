---
name: ionic-sentry
description: Wire Sentry for crash + error reporting in an Ionic Capacitor app via @sentry/capacitor + the framework-specific @sentry SDK. Trigger when adding crash reporting, error tracking, performance monitoring, or session replay to an Ionic app.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Sentry (Crash + Error Reporting)

`@sentry/capacitor` captures native iOS / Android crashes; the framework SDK (`@sentry/angular` / `@sentry/react` / `@sentry/vue`) captures JS errors, breadcrumbs, and performance traces. Install both — they integrate via a single `init()` call.

## When to consult

- **Setup, init, and per-framework integration**: [setup.md](references/setup.md)

## Hard rules

- ✅ Init Sentry **as the first thing in app bootstrap**, before any other code can throw.
- ✅ Set the `release` value to your app version (e.g., from `package.json`) so source maps / releases line up in the Sentry dashboard.
- ✅ Set `environment: 'production'|'staging'|'development'` from the build mode so you can filter.
- ✅ Upload **source maps** as part of the build pipeline — without them, JS stack traces are unreadable.
- ❌ Don't send PII (emails, full names, IPs) unless you've enabled the privacy controls and have legal sign-off.
- ❌ Don't init twice — `@sentry/capacitor` wraps the framework SDK; calling both init's separately double-reports everything.

## Library

```bash
npm install @sentry/capacitor @sentry/angular   # for Angular
npm install @sentry/capacitor @sentry/react     # for React
npm install @sentry/capacitor @sentry/vue       # for Vue
npx cap sync
```
