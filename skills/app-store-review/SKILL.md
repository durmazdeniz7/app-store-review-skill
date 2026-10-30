---
name: app-store-review
description: Audit an iOS/iPadOS/macOS app project against Apple's App Store Review Guidelines before submission and produce a pass/warn/fail report with fixes. Use when the user asks for an App Store review check, pre-submission audit, "App Store'a uygun mu", "red yer mi", rejection risk, or mentions guideline numbers like 2.1, 3.1.1, 4.8, 5.1.1.
---

# App Store Review Audit

Audit the current project against Apple's App Store Review Guidelines
(https://developer.apple.com/app-store/review/guidelines/) and report every
likely rejection risk with the guideline number, evidence (file:line) and a
concrete fix.

Write the final report in the language the user is writing in.

## 0. Before you start

1. If web access is available, fetch the guidelines URL above and note its
   "Last Updated" date. If any rule below conflicts with the live page, the
   live page wins; mention the difference in the report.
2. Ask the user only if you cannot infer it from the code:
   - Does the app sell anything (digital content, subscriptions, physical goods)?
   - Is it aimed at children / will it be in the Kids Category?
   - Which storefronts (US only, EU, worldwide)?
   If the user is not around, assume worldwide, not Kids Category, and say so.

## 1. Detect the project

Identify the stack and locate the key files. Use Glob/Grep, don't guess.

| Stack | Look for |
|---|---|
| Native (Swift/ObjC) | `*.xcodeproj`, `Info.plist`, `*.entitlements`, `PrivacyInfo.xcprivacy`, `Podfile`, `Package.swift` |
| React Native / Expo | `package.json`, `app.json` / `app.config.*`, `ios/` folder, `eas.json` |
| Flutter | `pubspec.yaml`, `ios/Runner/Info.plist` |
| Capacitor / Cordova / WebView | `capacitor.config.*`, `config.xml`, `www/` |

List all third-party SDKs (Podfile.lock, Package.resolved, package.json,
pubspec.lock). Many checks below depend on this list.

## 2. Automated checks (evidence from code)

Run every check. For each finding record: status, guideline, evidence, fix.

### Safety (1.x)
- **1.1.5** Religious texts or quotes in the app: check they're accurately
  sourced; flag any hard-coded religious quotes without a source.
- **1.2 User-generated / AI-generated content**: if users can post, share,
  chat, or the app generates content with AI, verify ALL of: content
  filtering, a "report" action, a "block user" action, and visible contact
  info. Missing any one = FAIL.
- **1.4.1 Medical / health**: if the app gives health info or measurements,
  look for a "consult a doctor" disclaimer and no unverifiable accuracy claims.
- **1.5 Developer info**: an in-app way to contact support (email, URL).
- **1.6 Data security**: hard-coded API keys/secrets in the client
  (grep for `sk-`, `api_key`, `secret`, `Bearer `, `AIza`), plain `http://`
  endpoints, `NSAllowsArbitraryLoads = true` in ATS settings.

### Performance (2.x)
- **2.1 Completeness**: grep for `lorem ipsum`, `TODO`, `FIXME`,
  `coming soon`, `yakında`, `test@`, `localhost`, `127.0.0.1`, staging URLs,
  debug menus, `print`/`console.log` of sensitive data. Any visible
  placeholder in UI strings = FAIL.
- **2.1(b) IAP visible**: every IAP product ID in code is reachable from UI.
- **2.3.7 Name**: display name (`CFBundleDisplayName` / app.json `name`)
  is 30 characters or fewer and has no keyword stuffing.
- **2.3.8** Words like "for kids" / "çocuklar için" in name or metadata
  when the app is not in the Kids Category = FAIL.
- **2.4.1 iPad**: check device family / `supportsTablet`; iPhone-only is
  allowed but flag as WARN and check the layout doesn't break on iPad if enabled.
- **2.4.4** Never ask the user to restart the device or change unrelated
  system settings.
- **2.5.1 Private APIs**: grep for `NSClassFromString`, `performSelector`,
  `dlopen`, `dlsym`, selectors/classes starting with `_`. Flag each.
- **2.5.2 Remote code**: downloading and executing code that changes app
  features (remote JS eval, dynamic frameworks). OTA updates (Expo Updates,
  CodePush) are WARN: OK only for bug fixes, not for new features.
- **2.5.4 Background modes**: every entry in `UIBackgroundModes` must be
  used for its real purpose (audio actually plays, location actually needed).
- **2.5.5 IPv6**: hard-coded IPv4 addresses = WARN.
- **2.5.14 Recording**: camera/mic/screen recording must have explicit
  consent and a visible indicator.
- **2.5.18 Ads**: ads only in the main app (not widgets/extensions/
  notifications), interstitials have a clear close button, there's a way to
  report an ad, no targeting with health or kids data.
- **2.5.19 Hidden features**: remote-config/feature flags that unlock
  undocumented functionality after review = FAIL.

### Business (3.x)
- **3.1.1 In-App Purchase**: if digital content/features are unlocked,
  verify StoreKit / RevenueCat / react-native-iap / in_app_purchase is used.
  Stripe, PayPal, iyzico, license keys, promo QR codes or external "buy on
  our website" links for digital goods = FAIL (except US storefront links
  and the entitlements in 3.1.1(a)). Physical goods/services consumed
  outside the app must NOT use IAP (3.1.3(e)).
- **Restore purchases**: a "Restore Purchases" button exists for
  non-consumables/subscriptions. Missing = FAIL.
- **3.1.2 Subscriptions**: paywall shows price, period, what's included,
  auto-renew notice, and links to Terms of Use (EULA) and Privacy Policy.
  Period at least 7 days. Free trial terms stated clearly.
- **3.1.2(a)** Users who already paid for a one-time unlock keep it if
  the app moves to subscriptions.
- **3.2.2(x) / 5.6.1 Reviews**: no gating features behind rating the app;
  rating prompts must use `SKStoreReviewController` / `requestReview`
  (or `expo-store-review`, `in_app_review`). Custom "rate us 5 stars"
  dialogs = FAIL.

### Design (4.x)
- **4.1 Copycats**: other brands' names/icons in app name, icon, or UI.
- **4.2 Minimum functionality**: app that is mostly a `WKWebView`/WebView
  loading one website = FAIL risk. Note which native features exist.
- **4.2.3** App works without installing another app; large initial
  downloads disclose size and ask first.
- **4.3 Spam**: multiple near-identical targets/bundle IDs in the project.
- **4.5.4 Push**: app works if push permission is denied; marketing pushes
  require opt-in and an opt-out.
- **4.8 Login**: if Google/Facebook/X/other social login is the primary
  account, an equivalent privacy-friendly option (e.g. Sign in with Apple)
  must also be offered. Missing = FAIL.

### Legal (5.x)
- **5.1.1(i) Privacy policy**: an in-app link to the privacy policy.
- **5.1.1(ii) Purpose strings**: for every permission API used, the matching
  `NS…UsageDescription` key exists in Info.plist AND clearly explains why
  (e.g. not just "Camera"). Check at least: Camera, Microphone, Photo
  Library (+Add), Location (WhenInUse/Always), Contacts, Calendars,
  Reminders, Bluetooth, Face ID, Motion, Speech, Health, Tracking.
  Key present for an unused permission = WARN; used without key = FAIL (crash).
- **5.1.1(iii)–(iv) Data minimization**: no permission requested that the
  core feature doesn't need; app still usable when a permission is denied.
- **5.1.1(v) Accounts**: login not forced when there are no account-based
  features. If the app supports sign-up, **in-app account deletion** must
  exist (not just "email us"). Missing = FAIL.
- **5.1.2(i) Third-party AI**: if user data is sent to OpenAI, Anthropic,
  Gemini or any other AI/LLM API, the app must clearly disclose it and get
  explicit permission before sending. No consent screen = FAIL.
- **5.1.2 Tracking (ATT)**: if IDFA/AdSupport, Facebook SDK, AppsFlyer,
  Adjust, or similar tracking SDKs are present, verify
  `ATTrackingManager.requestTrackingAuthorization` and
  `NSUserTrackingUsageDescription`. Features must not require allowing tracking.
- **Privacy manifest** (Apple upload requirement related to 5.1): a
  `PrivacyInfo.xcprivacy` exists and declares Required Reason APIs used by
  the app (UserDefaults, file timestamps, system boot time, disk space,
  active keyboards) and collected data types. Check that bundled SDKs ship
  their own manifests.
- **5.1.3 Health**: HealthKit data not used for ads; nothing health-related
  stored in iCloud.
- **5.1.4 Kids**: if aimed at children: no third-party analytics/ads,
  parental gate before links/purchases.
- **5.1.5 Location**: only used when directly relevant; explained to user.
- **5.2 Intellectual property**: third-party logos, fonts, images, music,
  or content (e.g. YouTube downloads, scraped data) without a license.
- **5.3 Gambling / 5.4 VPN / 5.5 MDM**: if present, flag as high-scrutiny
  and list the licensing/organization-account requirements.

## 3. Manual checklist (cannot be verified from code)

Always include this list in the report so the user can tick it off in
App Store Connect:

- [ ] Screenshots show the app in use (not just splash/login) and match the current version (2.3.3)
- [ ] Description, subtitle and keywords are accurate, no competitor names or prices (2.3.1, 2.3.7)
- [ ] Correct category and honest age-rating answers (2.3.5, 2.3.6)
- [ ] App Privacy ("nutrition label") answers match the SDKs and data actually collected (2.3, 5.1)
- [ ] Privacy Policy URL and Support URL filled in and working (1.5, 5.1.1)
- [ ] Demo account (or demo mode) and backend live during review (2.1)
- [ ] Review Notes explain non-obvious features and IAPs specifically (2.3.1)
- [ ] All IAP products created, in "Ready to Submit", with review screenshot (2.1(b))
- [ ] "What's New" text describes significant changes (2.3.12)
- [ ] Tested on a real device for crashes, including iPad if supported (2.1)
- [ ] Regulated field (finance, health, gambling)? Submitted by the legal entity, not an individual (5.1.1(ix))

## 4. Report format

Start with a one-line verdict: **Ready / Fix before submitting / High rejection risk**.

Then a table sorted by severity (FAIL first):

| Status | Guideline | Finding | Evidence | Fix |
|---|---|---|---|---|
| ❌ FAIL | 5.1.1(v) | No in-app account deletion | `AuthService.swift:42` (signup only) | Add Settings → Delete Account that deletes server data |
| ⚠️ WARN | 2.5.2 | OTA updates enabled | `app.json: updates.enabled` | Use only for bug fixes |
| ✅ PASS | 4.8 | Sign in with Apple offered alongside Google | `LoginView.swift:18` | — |

Rules for the report:
- Only mark PASS when you actually found evidence; otherwise "Not checked"
  with the reason.
- Never invent file paths or line numbers.
- Group checks that don't apply (e.g. no ads, no IAP) in one "N/A" line.
- End with the manual checklist and, if asked, offer to implement the fixes.
- Do not change code during the audit unless the user asks.
