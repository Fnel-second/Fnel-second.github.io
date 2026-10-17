# 🔄 Tesla Ready North America update guide (existing users)

For people already using a Tesla Ready North America bot. New users: follow the [main guide](https://fnel-second.github.io/tesla-ready-north-america/).

## ⚠️ First: don't import the template again

- Importing the template again most likely creates a **second bot** instead of updating yours (inferred, unverified).
- Then you'd have two sets of routines and webhooks, and climate could start twice or you'd get duplicate reports.
- Instead, tell your **existing bot**:

> Read this update guide and apply the changes: https://fnel-second.github.io/tesla-ready-north-america/update.html

The bot updates its own memory, routines and skill, and walks you through anything you have to do yourself (phone automation changes), one step at a time.

## If you already created a second bot

1. Keep **only one** (the one with more settings is recommended).
2. Pause or delete all routines (webhook, mail, calendar) on the bot you're not keeping.
3. Make sure the webhook URL and key in your phone automation belong to the bot you kept. If not, copy them again from its routine panel (visible on PC).
4. Send body `test` once and check that only the kept bot replies.

## What changed and what to do

### 2026-10-17: App after-blow and cancel verified on a real car

- The Tesla Ready app's after-blow (car Bluetooth disconnect → `{"event":"after_blow"}`) and cancel button (`{"event":"after_blow_cancel"}`) were verified on a real car on Oct 17, 2026 (KST): `after_blow` received at 11:07, `after_blow_cancel` at 11:19, then an actual after-blow fan run from 11:26 to 11:31 succeeded. `climate_keeper_mode` (Camp/Dog mode) was confirmed readable via the API.
- ⚠️ **If you moved to the Tesla Ready app, turn off your old MacroDroid (or other) parking AND after-blow macros**, so signals aren't sent twice.

### 1. Android: Tesla Ready app recommended (Android users)

The recommended Android method is now the **Tesla Ready app** (MacroDroid stays as the alternative).

1. Download the [Tesla Ready app APK](https://github.com/Fnel-second/tesla-ready-app-releases/releases/latest/download/TeslaReadyApp.apk) (v1.1.0) and install it as in [main guide section 4](https://fnel-second.github.io/tesla-ready-north-america/). On the Play Protect block screen tap **More details** → **Install anyway** (not OK). Location "Allow all the time", battery optimization off, notifications on.
2. Enter your current webhook URL and key (from the routine panel on PC). Pick your parking area on the map or save your current location.
3. Tap the test button and check for the confirmation message.
4. Once it works, **turn off your old MacroDroid (or other) parking macro and after-blow macro** so signals aren't sent twice.
5. After-blow users: choose the car's Bluetooth in the app so it sends `after_blow` on disconnect; the app's cancel button sends `after_blow_cancel`. Both were verified on a real car on Oct 17, 2026 (see below). Replying "cancel" in chat also works. If you keep a MacroDroid after-blow macro instead, leave the app's car Bluetooth empty.
6. The English display and US settings (OpenStreetMap, ft/mi radius) are unverified.

### 2. Webhook routine accepts JSON

The app sends JSON like `{"event":"parking_enter","place":"<name>"}`. The webhook routine must accept both a plain-text body and JSON with an `event` field (see "For the bot" below). Old plain-text signals keep working.

### 3. Weekly update check

Your bot checks this page and the app's latest release once a week and tells you when something changed. It applies changes only after you agree.

---

## 🤖 For the bot (what an existing bot should do after reading this)

Keep the owner's settings (webhook URL and key, time window, parking location, secrets) as they are. Never print secrets.

1. **Memory**: record that this update was applied (with the date), so you don't announce it again.
2. **Webhook routine**: parse the signal from either a plain-text body (`test`, `parking_enter`, `after_blow`, `after_blow_cancel`) or a JSON body's `event` field. For `parking_enter`, if JSON has `place`, include it in the report and ignore a repeat from the same `place` within 10 minutes; handle it the same without `place`. Keep the existing after-blow behavior (20-min cancel wait, chat "cancel", P and Camp/Dog check).
3. **Android users**: guide them through section 1 above, one step at a time. After the app test works, have them turn off the old MacroDroid (or other) parking and after-blow macros so signals aren't duplicated. App after-blow and cancel were verified on a real car on Oct 17, 2026. Ask Android or iPhone first if unknown.
4. **Duplicates**: if you see two routines doing the same job, tell the owner and help keep only one.
5. **Update check routine**: if it doesn't exist yet, create the weekly "Guide update check" routine (Monday morning). It reads this page and the app's latest release, stays silent if nothing changed since last time, otherwise summarizes the changes for the owner and applies only what they agree to. Never touch secrets.
