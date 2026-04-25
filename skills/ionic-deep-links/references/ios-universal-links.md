# iOS Universal Links

Universal Links let `https://yourapp.com/...` open the app directly. They require:

1. An entitlement in the app.
2. An `apple-app-site-association` (AASA) file hosted at your domain.

## App side

### Add the Associated Domains capability

`npx cap open ios` → select the App target → **Signing & Capabilities** → **+ Capability** → **Associated Domains** → add:

```
applinks:yourapp.com
```

For staging/dev: `applinks:staging.yourapp.com`. You can list multiple domains.

This writes into `App.entitlements`:

```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:yourapp.com</string>
</array>
```

### `Info.plist` — nothing extra

Universal Links don't need `CFBundleURLTypes` (that's custom schemes only).

## Domain side

### Host the AASA file

Place a file at:

```
https://yourapp.com/.well-known/apple-app-site-association
```

Note: **no `.json` extension** and **`Content-Type: application/json`**.

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.company.appname",
        "paths": [
          "/reset-password*",
          "/invite/*",
          "/share/*",
          "NOT /admin/*"
        ]
      }
    ]
  }
}
```

- `appID` = your Team ID (from Apple Developer portal) + bundle ID.
- `paths` = which URL paths open the app. Use `*` for wildcards. `NOT` excludes paths.

### Verify the AASA

```bash
curl -I https://yourapp.com/.well-known/apple-app-site-association
# Look for: HTTP/2 200, Content-Type: application/json
```

Apple's CDN caches the AASA. Use Apple's [Universal Links validator](https://search.developer.apple.com/appsearch-validation-tool) to refresh + verify it parses.

### Common problems

- **Server returns `text/html`**: use `Content-Type: application/json`.
- **Server requires the `.json` extension**: rename to `apple-app-site-association` (no extension) and configure server MIME type.
- **Redirect**: AASA must be served directly with a `200`. Redirects break Universal Links.
- **Behind auth**: the file must be public.
- **HTTPS only**: HTTP doesn't work.

## Test

After installing the app on a device (Universal Links don't fire in the iOS Simulator for non-development builds):

1. Long-press a link to `https://yourapp.com/reset-password?token=abc` in Notes or Messages.
2. Choose **Open in [App]**. If "Open in [App]" doesn't appear, the AASA didn't validate.

The `appUrlOpen` listener (see [custom-scheme.md](custom-scheme.md)) fires with the full `https://` URL — no special handling needed.

## What if a user has the app uninstalled?

Universal Links degrade gracefully — the user just lands on the web URL. Make sure that page exists and ideally promotes the app.
