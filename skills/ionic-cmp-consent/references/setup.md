# UMP Setup (AdMob console)

UMP is configured in the **AdMob console**, not in code — code only triggers the dialog the console publishes.

## In AdMob console

1. Sign in at <https://apps.admob.com>.
2. Navigate to **Privacy & messaging** → **GDPR**.
3. Click **Create message**.
4. Configure:
   - **App** — select the app(s) the message applies to.
   - **Languages** — at minimum, add the languages your app supports.
   - **Type** — "Consent" (TCF v2).
   - **Customization** — branding, color, logo.
5. **Publish** the message. AdMob's TCF strings then start serving in the targeted regions.

Optionally, also create a **CCPA** message under Privacy & messaging → US states for California users — same flow.

## What "publish" actually does

Publishing a UMP message tells Google's servers to start returning a `REQUIRED` consent status for users in the targeted regions. The Capacitor app then asks for that status at runtime and shows the form when needed.

A published message is live within minutes for new requests; existing app sessions won't see it until next launch.

## Test devices

Before going live, register test devices so the consent form appears for you regardless of region:

1. Run the app once on the device with debug enabled.
2. Find the device hash:
   - **iOS**: in the Xcode console, grep for `test device hash` — UMP logs `Use the following test device hash: <hash>` on first run.
   - **Android**: `adb logcat | grep -i "TAG_GMA"` — the SDK prints the hash on first run.
3. AdMob console → **Privacy & messaging** → **Settings** → **Test devices** → add the hash + select **Geography: EEA**.

This forces the form to render on the test device every time, useful for QA.
