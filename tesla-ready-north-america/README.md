# 🚗 Tesla Ready North America: Auto-start your Tesla's climate on the way out

Walk into your parking garage and your Tesla's climate turns on by itself. On days with a calendar event, it checks live traffic and starts climate right on time for your departure.
Built with Grok Bot, the official Tesla Fleet API, Android MacroDroid or iPhone Shortcuts, and the Google Maps Platform.

👉 Template: [Tesla Ready North America template](TEMPLATE_LINK_TBD)
(After you import the template, the bot walks you through the setup below in your first chat, one question at a time.)

> ⚠️ **Community edition.** This North American edition is adapted from the Korean version and has not been tested by the author in North America. If a screen, error, or step doesn't match this guide, ask your bot to troubleshoot it with you. It will check the official Tesla and Google docs and work through fixes step by step.

---

## How it works

```
[Phone: MacroDroid / iPhone Shortcuts] enters parking area
   → HTTP POST (webhook)
   → [Grok Bot routine] Is it within your commute window? Already started today?
   → [Bot's computer] refresh token → command-signing proxy
   → [Tesla Fleet API] wake vehicle → start climate → verify it's on
```

For calendar-based prep, the bot also sends the event location to your car's navigation when it starts climate (section 5). The parking-webhook commute trigger has no destination, so it sends no navigation.

- Typical result: climate on within about a minute of the signal (based on the Korean edition; not yet measured in North America).
- Climate starts at most once per day. Signals outside your window are ignored.
- Official API only (no Tessie, TeslaFi, or other third parties). Third-party services charge a monthly fee and hold your Tesla account token. With the official API, cost is close to $0 for personal use, and your keys and tokens stay on your bot's computer.

## What you need

- A Tesla and a Tesla account
- Grok Bot
- A phone automation app: MacroDroid on Android, or the built-in Shortcuts app on iPhone (allow Shortcuts location access "Always")
- Gmail (only if you send signals by email instead of HTTP)
- A GitHub account (GitHub Pages hosts your public key)
- A Google Cloud account with billing enabled (only for calendar-based prep)
- (Optional, recommended) A calendar connected to Grok Bot, e.g. Google Calendar (required for calendar-based prep)

---

## 🔰 New to APIs?

Difficulty is medium-high, but Grok Bot does most of the technical work on its own computer. After importing the template, just tell the bot "walk me through it from the start."

**What the bot does**
- Generates the key pair (private and public key)
- Installs and runs `tesla-http-proxy`
- Writes the token-refresh, wake, and climate scripts
- Sends the partner registration request and builds the webhook routine
- Checks test results and diagnoses errors

**What you do** (mostly clicking)
1. Sign up for GitHub → create a repo named `<username>.github.io` → enable Pages → add an empty `.nojekyll` file at the repo root (otherwise the `.well-known` folder isn't served). Recommended: personal GitHub Pages, so `/.well-known/` sits at your domain root. Cloudflare Pages or Netlify may also work (unverified)
2. Upload the public key file the bot gives you to `.well-known/appspecific/` in that repo (the bot tells you the file and path)
3. Sign in at developer.tesla.com → create an app → enter the origin, redirect URL, and scopes the bot gives you
4. Give the bot your Client ID. Enter the Client Secret only in the bot's secret-entry card (never paste it into chat)
5. Open the auth link from the bot and approve with your Tesla account
6. Approve the virtual key in the Tesla app or on the car's screen
7. Confirm a payment method on the Tesla developer billing page
8. (Calendar prep only) Create a Google Maps Platform API key and enter it via the bot's secret-entry card
9. Build the phone automation: an Android MacroDroid macro or an iPhone Shortcuts personal automation (section 4)

If you get stuck, send the bot a screenshot of that screen. Sections 1–3 below explain what the bot does, for the curious.

---

## 1. Create a Tesla Fleet API app

1. Create an app at developer.tesla.com (your Tesla account needs a verified email and multi-factor authentication). Requests are auto-rejected if the app name already exists; names containing `tesla` were also rejected in the Korean edition's experience (unverified for NA).
2. Generate an EC key pair (prime256v1) and publish the public key on GitHub Pages at `/.well-known/appspecific/com.tesla.3p.public-key.pem`. The private key stays on the bot's computer only.
3. Register the allowed origin (`https://<username>.github.io`) and redirect URL (`.../callback`).
4. Register the partner account (`POST /api/1/partner_accounts` with a partner token) against the North America / Asia-Pacific base URL: `https://fleet-api.prd.na.vn.cloud.tesla.com` (per Tesla's Regions and Countries doc). Registration is per region; calling the wrong region returns 412 or 421. The domain must match the root domain of your allowed origin.
5. OAuth scopes: `openid offline_access vehicle_device_data vehicle_cmds vehicle_charging_cmds vehicle_location`
   If you leave out `vehicle_location`, you'll have to re-authorize later.
6. Add the virtual key: open `https://www.tesla.com/_ak/<username>.github.io` on your phone (Tesla app opens) and follow the prompts.

💰 Pricing (developer.tesla.com, checked Oct 2026): $1 per 1,000 commands, $1 per 500 data requests, $1 per 50 wakes, with a $10 monthly discount per account. Tesla says the discount roughly covers 100 commands and 2 wakes per day for two vehicles, so personal use should be $0. Apps without a payment method are automatically disabled, and the default billing limit is $0, so add a payment method and set a small billing limit (e.g. $5) on the billing and usage page. Payment is available in the US, CA, MX, and PR.

## 2. Command-signing proxy and scripts

Recent Teslas only accept signed commands.

- Run `tesla-http-proxy` from Tesla's official `vehicle-command` repo on `127.0.0.1:4443`.
- The script does the following, in order:
  1. Starts the proxy if it's not running (handles reboots).
  2. Refreshes the access token with the refresh token.
  3. Sends `wake_up`. It must include the body `{}` and `Content-Type: application/json` (an empty body returns 406).
  4. Waits up to about 1 minute for the car to come online.
  5. Sends `set_temps` (your temperature, converted to °C for the API) and `auto_conditioning_start` through the proxy.
  6. Checks `climate_state` to confirm climate is on.
- A state file records `last_on_date` so climate starts only once per day.

Note: the Fleet API takes temperatures in °C. The bot stores your setting in °F (default 70°F ≈ 21°C) and converts it.

## 3. Grok Bot webhook routine

- Trigger: webhook
- The body tells signals apart:
  - `test`: just confirms receipt
  - `parking_enter`: commute climate
  - `after_blow`: after-blow (section 6, only if enabled)
  - `after_blow_cancel`: cancels a pending after-blow
- For `parking_enter`: if it's a weekday between 5:30 PM and 11:00 PM (change to suit you) and climate hasn't started today, the script runs. On failure it retries once.
- The routine panel shows the webhook URL and key. If you're setting up from the Grok Bot phone app, the URL and key are currently only visible on PC, so copy them on a PC and move them to your phone. Delete any copies you used to move them (messages, email, etc.).

## 4. Phone automation (Android MacroDroid / iPhone Shortcuts)

| Item | Android (MacroDroid) | iPhone (Shortcuts) |
|---|---|---|
| Where | Add Macro | Shortcuts app > Automation > Personal Automation |
| Parking trigger | Location (geofence), enter parking area | "Arrive" (location: parking area, set radius) |
| Run automatically | Runs immediately by default | Turn on "Run Immediately", choose "Notify When Run" ¹ |
| Webhook action | HTTP Request | "Get Contents of URL" |
| Method / URL | `POST`, `<webhook URL>` | Method `POST`, URL `<webhook URL>` |
| Header | `Authorization` = `Bearer <webhook key>` | `Authorization` = `Bearer <webhook key>` |
| Body | Content type `text/plain`, body `parking_enter` | Request Body "Text" `parking_enter` (if Text isn't offered, JSON `event` = `parking_enter`) ¹ |
| Permission | MacroDroid location "Allow all the time" | Settings > Privacy & Security > Location Services > Shortcuts "Always" ¹ |
| Check result | Save HTTP response code to local variable `code` → toast `{lv=code}` | Add "Show Notification" to display the result ¹ |

¹ Menu names may differ by iOS version. The iPhone method has not been tested directly.

- ⚠️ On Android, if you don't set the content type you get a 415 error.
- (Optional) Also send an email in the same automation as a backup signal. Email is checked every 5 minutes, so it's slower.

### If HTTP requests are hard (Gmail method)

Instead of a webhook, you can signal by email. Grok Bot checks mail every 5 minutes, so climate can start up to 5 minutes late.

1. Change the phone action to send an email (To: your Gmail address, Subject: `[Tesla] parking_enter`).
   - Android: MacroDroid action "Send Email"
   - iPhone: Shortcuts action "Send Email", turn off "Show Compose Sheet" (name may differ by iOS version). If you can't turn it off, it may not send automatically (unverified).
2. Create a Gmail filter: from your own address, subject contains `[Tesla]` → Skip the Inbox, apply label `Tesla`. Keeps your inbox clean.
3. Connect the Gmail connector to Grok Bot.
4. Tell the bot "set up the commute climate mail routine." During your weekday window it checks every 5 minutes for new mail with subject `[Tesla] parking_enter` and starts climate if found. Processed mail is recorded so climate won't start twice.
5. To test, send yourself an email with that subject.

You can use webhook and email together. Climate starts only once per day, so the first signal wins and the rest are ignored.

## 5. Calendar-based prep (optional)

### 📅 Connect a calendar (recommended)

With a calendar plugin (e.g. Google Calendar) connected, the bot reads your events and preps on its own.
- Event start time = arrival deadline.
- The event's "Location" is the routing destination. Put an exact address or venue name there.
- Without a calendar, calendar-based prep doesn't run automatically. You'd ask directly, e.g. "arrive at ___ by 12:30 PM Saturday."

On event days, the bot gets drive time from the car's live location to the event location using the Google **Routes API** (`computeRoutes`, `travelMode: DRIVE`, `routingPreference: TRAFFIC_AWARE`, departure time = now).

- Car departs = event start − buffer after arrival − drive time
- You leave = car departs − walk time to the car
- Climate on = car departs − 10 min

It recalculates 2 hours, 1 hour, and 30 minutes before departure, and moves climate and departure earlier if traffic gets worse.

### 🧭 Auto-send destination to navigation

When climate starts, the bot sends the destination (its own parking, if found) to the car's navigation, so you can drive off right away.
1. Read the destination from the event's "Location".
2. Get coordinates with the Google **Geocoding API**.
3. Look for the destination's own parking with the Google **Places API (New) Text Search**:
   - `POST https://places.googleapis.com/v1/places:searchText`, body `{"textQuery": "<venue name> parking"}`, header `X-Goog-Api-Key`, field mask `places.displayName,places.formattedAddress,places.location`
   - Use the venue/building name, not the address string. If the location is only an address, use the building name from geocoding; if there's none, skip the parking search and use the original address.
   - Candidates: keep only places whose name contains the venue name (case and spaces ignored). Example: for "Example Hospital", keep "Example Hospital Parking" or "Example Hospital Garage B", but drop nearby public or other buildings' lots even if closer.
   - A 0.3-mile (~500 m) distance check is only a secondary filter on name-matched candidates.
   - ⚠️ If no parking is listed under that name, the bot uses the original address.
   - This search and choice happen when the bot first plans the trip, not when climate starts.
     - 1 candidate: used without asking.
     - Several (up to 5): the bot shows names and addresses, you pick, and the choice is saved in routine state.
     - No answer by climate time: the closest candidate to the destination is used, and the report says so.
     - 0 candidates: original address coordinates.
4. Send `navigation_gps_request` `{"lat": <lat>, "lon": <lon>, "order": 0}` through the signing proxy (the chosen parking coordinates, or the address coordinates).
5. If that fails, send the address with `share`: `{"type": "share_ext_content_raw", "locale": "en-US", "timestamp_ms": <now ms>, "value": {"android.intent.extra.TEXT": "<address>"}}`
6. The climate report includes the destination type (chosen parking / closest parking after no answer / original address) and the command used (navigation_gps_request / share / failed).
7. When you get in, check that the destination on screen is right.

**Google Maps Platform key (required for calendar-based prep)**: console.cloud.google.com → create a project → enable billing → enable **Geocoding API**, **Routes API**, and **Places API (New)** → Credentials → Create API key → restrict it to those 3 APIs. Enter the key only via the bot's secret-entry card (never paste it into chat).

💰 Google pricing (Google Maps Platform pricing page, checked Oct 2026; a billing account is required). Each SKU has a monthly free usage cap:

| What the bot uses | SKU | Free per month | After that (per 1,000) |
|---|---|---|---|
| Geocoding | Geocoding (Essentials) | 10,000 | $5 |
| Drive time with traffic (`TRAFFIC_AWARE`) | Compute Routes Pro | 5,000 | $10 |
| Parking search with location field | Places Text Search Pro | 5,000 | $32 |

A few events a day with 3–4 recalculations is a few hundred requests a month, well within the free caps (estimate). Set a budget alert and per-API quota caps in Google Cloud.

- Required Tesla scope is `vehicle_cmds`, same as climate. No extra auth.
- Not used for the parking-webhook commute climate.

## 6. After-blow (optional, off by default)

⚠️ Tesla has no official after-blow feature. This is a workaround with a similar effect: briefly running the fan after you park to dry moisture in the A/C system.

**Flow**
1. Signal received (`after_blow`) → doesn't start right away. Notifies your phone/chat: "After-blow starts in N min" (N = cancel wait, default 20 min).
2. Saves the pending state and start time to a state file and waits N minutes. If a cancel signal arrives, it skips.
3. Right before starting, rechecks via the Fleet API: in Park (`shift_state`) and parked, and Camp/Dog mode off (`climate_keeper_mode` is `off`). Otherwise it stops.
4. `set_temps` close to or slightly above the outside temperature
5. `auto_conditioning_start`
6. After the set fan time (5–10 min), `auto_conditioning_stop`. If the car leaves Park while running, stops immediately.
7. Restores your usual temperature setting.

**Passenger safety**
- The signal fires when the driver's phone disconnects from the car's Bluetooth. It could run with a passenger still inside, so there's a cancel wait.
- The API can't tell if someone is in the car (unverified).
- Whether `climate_keeper_mode` can be read via the API needs confirmation (unverified).
- To cancel: webhook body `after_blow_cancel`, email subject `[Tesla] after_blow_cancel`, or reply "cancel" in chat.

| Cancel button | Android (MacroDroid) | iPhone (Shortcuts) |
|---|---|---|
| Build | New macro, trigger "Shortcut Launched" (home screen shortcut/widget) → HTTP Request body `after_blow_cancel`, or email subject `[Tesla] after_blow_cancel` | New shortcut "Get Contents of URL" body `after_blow_cancel` (or email subject `[Tesla] after_blow_cancel`) → Add to Home Screen or widget ¹ |

- No daily limit; works after every drive. Duplicate signals (e.g. within 30 min) are ignored.

**Sending the signal**

| Item | Android (MacroDroid) | iPhone (Shortcuts > Automation > Personal Automation) |
|---|---|---|
| Trigger | "Bluetooth Disconnected" → your car | "Bluetooth" disconnect (choose car) or "CarPlay" disconnect ¹, "Run Immediately" on, "Notify When Run" |
| Webhook | HTTP Request (as in section 4), body `after_blow` | "Get Contents of URL" (as in section 4), body `after_blow` |
| Email | "Send Email", subject `[Tesla] after_blow` | "Send Email", subject `[Tesla] after_blow`, compose sheet off ¹ |

¹ Names may differ by iOS version.
- Email can be up to 5 minutes late, which is fine for after-blow (estimate).

**Notes**
- The API can't switch recirculation/fresh air. For best effect, leave the car on Auto or fresh air (unverified).
- Effect and cost aren't verified yet. It's a few commands per run, likely within Tesla's monthly credit (estimate).

---

## Testing

1. Send body `test`. A confirmation message means it's connected.
   - Android: run the macro manually in MacroDroid and check the response code toast
   - iPhone: run a shortcut with the same action manually and check "Show Notification" (names may differ by iOS version)
2. Send `parking_enter` within your window to actually start climate.

## Common errors

| Symptom | Fix |
|---|---|
| HTTP 415 | Set body content type to `text/plain` or `application/json` |
| 401 / 403 | Check header `Authorization: Bearer <key>` |
| 404 | URL may be cut off; copy it again |
| Response code 0 | Check phone network and MacroDroid permissions |
| iPhone arrival doesn't fire | Location Services: Shortcuts "Always", "Run Immediately" on, widen the arrival radius |
| iPhone email doesn't auto-send | Turn off "Show Compose Sheet"; otherwise use the webhook |
| Wake 406 | Add body `{}` and Content-Type |
| Google `REQUEST_DENIED` / 403 | Billing enabled? API enabled? Key restrictions include that API? |
| Wrong parking chosen | Only lots named after the venue qualify, so put the exact venue name in the event location. Still wrong: shrink the distance or turn off parking-first |
| Nav destination doesn't appear | Check geocoding result → then the share result; fix the event location to an exact address |
| Fails after reboot | Restart the proxy (the script does this automatically) |
| After-blow fires on an unrelated Bluetooth drop | Cancel during the wait. The Park check also filters it. If frequent, limit the trigger to the car or use CarPlay disconnect |
| Temperature not restored after after-blow | Check restore temperature; check the report for a failed `auto_conditioning_stop` |

## Values to customize

| Item | Example / notes |
|---|---|
| Climate temperature | Default 70°F (set each time climate starts) |
| Units | °F and miles (default) |
| Commute days and window | e.g. weekdays 5:30–11:00 PM |
| Parking location (geofence) | Android: MacroDroid location trigger / iPhone: Shortcuts "Arrive" location and radius |
| Walk time to car | e.g. home 2 min, work 15 min |
| Climate lead time | e.g. 10 min before car departs |
| Auto-send nav destination | On / off (default on, calendar prep only) |
| Parking-first destination | On / off, distance (e.g. 0.3 mi) |
| Calendar | (Recommended) Google Calendar etc. |
| Tesla Client ID/Secret, key pair | Kept only on the bot's computer |
| GitHub Pages address, vehicle | Your own |
| After-blow | On / off (default off) |
| After-blow fan time | e.g. 5–10 min |
| After-blow cancel wait | Default 20 min |
| Restore temperature after after-blow | e.g. your usual setting (70°F) |
| Webhook URL and key, Google API key | Never share |

🔒 Never post tokens, keys, or webhook URLs anywhere public.
