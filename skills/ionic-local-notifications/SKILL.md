---
name: ionic-local-notifications
description: Schedule local (on-device) notifications via @capacitor/local-notifications — reminders, daily prompts, post-action follow-ups. Trigger when adding scheduled reminders, drink-water-style daily nudges, or any notification fired by the app itself rather than a server. Distinct from push notifications.
license: MIT
metadata:
  author: erkamyaman
  version: '1.0'
---

# Local Notifications

Notifications scheduled and fired **by the device**, not by a backend. Use cases: daily reminders, follow-up prompts after onboarding, post-purchase nudges, inactivity reminders.

For server-sent notifications (FCM / APNs), see [`../ionic-shared/references/push-notifications.md`](../ionic-shared/references/push-notifications.md). The two plugins are independent and can be used together.

## When to consult

- **Setup, permissions, scheduling**: [setup-and-scheduling.md](references/setup-and-scheduling.md)
- **Patterns** (recurring, channels, action handling): [patterns.md](references/patterns.md)

## Hard rules

- ✅ Request permission **when the user opts in** to a feature that uses notifications, not on app launch.
- ✅ Always set a unique numeric `id` per notification — same `id` overwrites the previous one.
- ✅ On Android 13+ the OS auto-prompts for `POST_NOTIFICATIONS` the first time you schedule.
- ❌ Don't schedule more than ~64 simultaneous notifications on iOS — the OS silently drops the rest.
- ❌ Don't rely on local notifications for time-critical delivery — they can be delayed by the OS for power management. Push is for "this must arrive."

## Library

```bash
npm install @capacitor/local-notifications
npx cap sync
```
