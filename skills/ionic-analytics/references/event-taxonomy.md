# Event Taxonomy

Consistent event names matter more than which tool you pick. A small, stable taxonomy is worth more than a sprawling one.

## Naming

- **`snake_case`**, present-tense verb form: `paywall_viewed`, `subscribe_clicked`, `purchase_completed`.
- **Object first, action last**: `paywall_viewed`, not `viewed_paywall`. Sorts naturally in dashboards.
- **Past tense for outcomes**: `purchase_completed`, `signup_failed`. Present-progressive for lifecycle: `app_backgrounded`.

## Properties

- **Stable keys**, snake_case: `source`, `plan`, `error_code`.
- **Enum values when possible**: `plan: 'weekly' | 'yearly'`, not free text.
- **Don't overload one event**: split `purchase_completed` (success) from `purchase_failed` (with `error_code`).
- **Avoid PII**: no raw email, IP, full name. Hashed identifiers OK if necessary.

## Recommended starter set for the scaffold

### App lifecycle

- `app_opened` — on every launch (auto in most SDKs).
- `app_first_open` — first launch ever.
- `app_backgrounded`, `app_foregrounded`.

### Onboarding

- `onboarding_started`
- `onboarding_step_viewed` (props: `step`, `total`)
- `onboarding_completed`
- `onboarding_skipped`

### Paywall

- `paywall_viewed` (props: `source`)
- `plan_selected` (props: `plan`)
- `subscribe_clicked` (props: `plan`)
- `purchase_completed` (props: `plan`, `price`, `currency`)
- `purchase_failed` (props: `plan`, `error_code`)
- `purchase_cancelled` (props: `plan`)
- `restore_clicked`
- `restore_succeeded`
- `restore_failed` (props: `error_code`)

### Auth (if applicable)

- `signup_started` (props: `method` = 'email' | 'google' | 'apple')
- `signup_completed`
- `signup_failed` (props: `method`, `error_code`)
- `login_started`, `login_completed`, `login_failed`
- `logout`

### Settings

- `setting_changed` (props: `setting` = 'theme' | 'language' | 'notifications', `value`)
- `notifications_enabled`, `notifications_disabled`
- `onboarding_reset` (QA / power-user signal)

### Errors

- `error_shown` (props: `surface`, `error_code`, `error_message`)

## User properties (set, not events)

Sticky attributes that describe the user, set once and updated when they change:

- `plan` — `'free' | 'weekly' | 'yearly'`
- `language` — `'en' | 'tr'`
- `theme_preference` — `'light' | 'dark' | 'system'`
- `notifications_enabled` — `true | false`
- `signup_method` — `'email' | 'google' | 'apple'`
- `signup_date` — ISO string

These let you slice events by cohort: "did the new paywall convert better for `tr` users on `dark` theme?".

## Versioning

Don't append `_v2` to event names. Instead, add a property:

```typescript
analytics.capture('paywall_viewed', {
  source: 'onboarding',
  paywall_version: 'v3',     // schema-versioned property
});
```

This way historical events stay queryable and dashboards don't break when you iterate.

## Document it

Keep an `analytics-events.md` (or a Notion page) listing every event, its properties, and the surface it fires from. Without this, the taxonomy drifts and the data becomes noise within months.
