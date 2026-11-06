# Kin Care Connect

**Caring Together. Living Safer.**

Kin Care Connect is an Android app (Flutter + Firebase) that helps families support elderly relatives through routine activity awareness, plain-language medical explanations, scam warnings, family care coordination, care reminders and emergency contacts.

> **Status: production-oriented source code, not a published app.** The code compiles and passes static analysis and automated tests (see *Verification* below). To run it you must connect **your own** Firebase project, add **your own** AI/speech API keys as server secrets, and create **your own** signing key. Nothing here pretends those services already work.

---

## Contents

| Path | What it is |
|---|---|
| `lib/` | Flutter app (Dart, Material 3, clean architecture: `core/` + `features/<module>/{data,domain,presentation}`) |
| `test/` | Unit tests (validators, scam rules, reminder recurrence) and widget tests (buttons, onboarding, login) |
| `functions/` | Cloud Functions backend (TypeScript, 2nd gen): AI, speech, scam, family, reminders, activity, account, retention |
| `firestore.rules` / `storage.rules` | Security rules enforcing ownership, family membership, roles and consent |
| `firestore.indexes.json` | Composite indexes for every query the app makes |
| `firebase.json` | Firebase CLI config incl. emulators |
| `rules-tests/` | Security-rules tests (Firestore + Storage emulators, `@firebase/rules-unit-testing`, mocha, TypeScript) |
| `.github/workflows/ci.yml` | CI: analyze, test, debug APK, functions build/test, signed release on `main` |
| `docs/SECURITY_AND_PRIVACY.md` | Completed Section 8: security & privacy design |
| `docs/TESTING_AND_QA.md` | Test strategy, manual QA checklist, release checklist |
| `docs/ARCHITECTURE.md` | Modules, data model, endpoint map |
| `docs/COMPLIANCE_CHECKLIST.md` | Compliance / privacy / accessibility pass: what was checked, fixed, and what you still must do |
| `docs/DATA_INVENTORY.md` | Every personal-data field: purpose, retention, who sees it |
| `docs/THIRD_PARTY_SDKS.md` | Every dependency: data touched, off-device transfer, licence (no analytics/ads) |
| `docs/ASSET_LICENSES.md` | Source and licence of every image/icon |
| `docs/store-listing.md` | Honest Play Store copy + Data safety form answers |
| `docs/legal/` | Privacy Policy, Terms of Service, Cookie & Local Storage Policy, account-deletion page (**templates — need lawyer review**), generated from the in-app texts |
| `lib/core/config/business_info.dart` | **Your business details — fill the `YOUR_…` placeholders** |
| `tool/export_legal.dart` | Regenerates `docs/legal/*.md` from the in-app legal texts |

### Modules (brief Section 5)

A Splash · B Onboarding · C Authentication · D Profile · E Dashboard (bottom nav: Home, Family, Activity, Notifications, Profile) · F Routine Monitor · G Medical Translator · H Scam Shield · I Family Care · J Care Reminders · K Emergency Contacts · L Notifications · M Settings. All are implemented with real navigation, Firebase Auth, Firestore reads/writes, validation, and loading/error/empty states.

### Backend endpoints (brief Section 7)

The backend uses **Firebase callable functions**, which automatically verify the user's Firebase ID token and App Check token. Each maps to an endpoint in the brief:

| Brief endpoint | Callable |
|---|---|
| POST /api/medical/transcribe | `medicalTranscribe` |
| POST /api/medical/summarize | `medicalSummarize` |
| POST /api/medical/explain | `medicalExplain` |
| POST /api/scam/analyze | `scamAnalyze` |
| POST /api/family/invite | `familyInvite` (+ `familyRevokeInvitation`) |
| POST /api/family/accept-invitation | `familyAcceptInvitation` |
| GET /api/activity/summary | `activitySummary` |
| POST /api/reminders | `remindersCreate` |
| (Settings) data export / delete account | `accountExportData`, `accountDelete` |
| (scheduled) | `dispatchDueReminders` (every 5 min), `purgeExpiredData` (daily) |

---

## 0. Before you publish: fill in your business details

All legal texts, the About & contact screen and the store listing read from **one file**, `lib/core/config/business_info.dart`. Replace every placeholder, then run `dart run tool/export_legal.dart` (a test fails if `docs/legal/` is out of date). Until you do, the About & contact screen shows a "Setup incomplete — do not publish" warning.

| Placeholder | What to put there |
|---|---|
| `YOUR_COMPANY_NAME` | Brand / trading name of the operator |
| `YOUR_LEGAL_ENTITY_NAME_AND_REGISTRATION` | Registered entity and number (e.g. Pvt Ltd + CIN), or your own name if you operate it personally |
| `YOUR_REGISTERED_POSTAL_ADDRESS` | Registered postal address |
| `YOUR_SUPPORT_EMAIL` | Support, data-request and deletion email |
| `YOUR_WEBSITE_URL` | Your website |
| `YOUR_PUBLIC_PRIVACY_POLICY_URL` | Where you publish `docs/legal/privacy-policy.md` (required by Google Play) |
| `YOUR_GRIEVANCE_OFFICER_NAME` / `YOUR_GRIEVANCE_OFFICER_EMAIL` | Grievance Officer (DPDP Act) |
| `YOUR_GRIEVANCE_RESPONSE_TIME` | Your committed response time (confirm against the DPDP Rules with your lawyer) |
| `YOUR_CITY_FOR_JURISDICTION` | Courts for the Terms' governing-law clause |
| `YOUR_AI_PROVIDER_NAME` / `YOUR_AI_PROVIDER_PROCESSING_COUNTRY` | The AI/speech provider you configure in `functions/.env` and where it processes data |
| `YOUR_FIREBASE_DATA_LOCATION` | Firestore/Storage location you chose (e.g. asia-south1, Mumbai) |
| `YOUR-PROJECT` (AndroidManifest.xml) | Firebase project for the optional password-reset deep link (see §4) |

Also: replace the default Flutter launcher icon (it shows the Flutter logo, see `docs/ASSET_LICENSES.md`), have a lawyer review `docs/legal/`, publish the privacy policy and `docs/legal/account-deletion.md`, and complete the Play Data safety form from `docs/store-listing.md`.

## Privacy and compliance at a glance

- **Free:** no ads, in-app purchases, subscriptions or fees; AI costs are paid by the operator.
- **No cookies, no analytics, no ad SDKs.** Firebase Analytics is not included; the manifest also disables analytics, ad-ID collection and FCM auto-init.
- **First-run Privacy choices** (not a cookie banner): optional notifications and AI processing, nothing pre-ticked, equal Accept/Decline, stored with version + timestamp, changeable in Settings.
- **18+ only:** separate unticked age and Terms+Privacy checkboxes at sign-up, recorded with version and server timestamp.
- **Minimal data:** no account phone number, no DOB, no location/contacts/camera permission. See `docs/DATA_INVENTORY.md`.
- **Notifications:** every category switchable plus a one-tap "All notifications" switch. No marketing email exists.
- **Deletion and export** in-app, plus deletion by email without the app.
- **Accessibility:** labelled images and icon buttons, tested contrast in light/dark/high-contrast themes, visible focus outlines, keyboard/D-pad traversal, ≥48dp targets.

## 1. Prerequisites

- Flutter **stable** (built and tested against 3.47.9 / Dart 3.13.7) and Android Studio with an Android SDK (API 35+) and JDK 17
- Node.js **22** and the Firebase CLI: `npm i -g firebase-tools`
- FlutterFire CLI: `dart pub global activate flutterfire_cli`
- A Google account with billing enabled on Firebase (**Blaze plan** is required for Cloud Functions, Secret Manager and scheduled functions)
- An account with an AI provider offering an OpenAI-compatible API (chat + audio transcription). Pick a plan that **does not train on your data** and offers suitable data-processing terms for health information.

## 2. Create and connect your Firebase project

1. In the [Firebase console](https://console.firebase.google.com) create a project (e.g. `kin-care-connect`). Choose a Firestore location close to your users (for India: `asia-south1`, Mumbai).
2. Enable **Authentication → Sign-in method → Email/Password**.
3. Create **Firestore Database** (production mode) and **Storage**.
4. From the repo root:
   ```bash
   firebase login
   firebase use --add            # pick your project; creates .firebaserc
   flutter pub get
   flutterfire configure --platforms=android --android-package-name=com.kincareconnect.app
   ```
   This **replaces** the placeholder `lib/firebase_options.dart`. (Firebase client options are public identifiers, not secrets. Security comes from rules, App Check and server checks.)
5. **App Check:** console → App Check → register the Android app with **Play Integrity**. For debug builds, run the app once, copy the *debug token* printed in logcat and add it under App Check → Manage debug tokens.
6. **Cloud Messaging:** nothing extra is needed for Android.

## 3. Set backend secrets (never in the app)

```bash
cd functions
cp .env.example .env          # non-secret config: AI_BASE_URL, AI_MODEL, STT_*, region
firebase functions:secrets:set AI_API_KEY     # paste your language-model API key
firebase functions:secrets:set STT_API_KEY    # paste your speech-to-text key (can be the same key)
npm ci
npm test                      # builds and runs backend unit tests
```

If the secrets are missing, the app shows *"The AI service has not been set up yet"*. It never fakes a result.

## 4. Deploy rules, indexes and functions

```bash
firebase deploy --only firestore:rules,firestore:indexes,storage
firebase deploy --only functions
```

Index builds can take a few minutes. Until they finish, some lists show a loading error.

**Region:** functions deploy to `asia-south1` by default. If you change `FUNCTIONS_REGION` in `functions/.env`, pass the same value to the app: `--dart-define=FUNCTIONS_REGION=<region>`.

**Password reset links (optional deep link):** Authentication → Templates → Password reset → *Customize action URL* to `https://<your-project>.firebaseapp.com/reset-password`, then replace `YOUR-PROJECT` in `android/app/src/main/AndroidManifest.xml`. Without this, the standard Firebase web page handles resets, which also works.

## 5. Run the app

```bash
flutter run                    # on a device/emulator
# Against local emulators (Java required for the Firestore emulator):
firebase emulators:start
flutter run --dart-define=USE_EMULATORS=true --dart-define=EMULATOR_HOST=10.0.2.2
```
For emulator testing of callables, set `ENFORCE_APP_CHECK=false` in `functions/.env.local` only.

## 6. Build a signed release APK

1. Create an upload key (keep it safe and backed up; losing it blocks updates):
   ```bash
   keytool -genkey -v -keystore ~/kincare-upload.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
   ```
2. Create `android/key.properties` (git-ignored):
   ```properties
   storePassword=<your store password>
   keyPassword=<your key password>
   keyAlias=upload
   storeFile=/absolute/path/to/kincare-upload.jks
   ```
3. Build:
   ```bash
   flutter build apk --release          # build/app/outputs/flutter-apk/app-release.apk
   flutter build appbundle --release    # for Google Play
   ```
4. Add the SHA-1/SHA-256 of your upload key (and later the Play App Signing key) to the Firebase Android app settings.

Without `key.properties`, release builds fall back to debug signing so `flutter run --release` still works locally. **Do not distribute those.**

## Rules tests

Security-rules tests live in `rules-tests/` (a separate npm package: TypeScript, mocha, `@firebase/rules-unit-testing` v5, Firebase JS SDK v12, `firebase-tools` as a dev dependency). They run against the **Firestore and Storage emulators** with the demo project id `demo-kin-care-connect`, so no real Firebase project, login or credentials are needed.

Requirements: **Java 21+** (for the emulators) and **Node 22**.

```bash
cd rules-tests
npm ci                    # first time
npm test                  # starts Firestore + Storage emulators, builds, runs all 325 tests, stops the emulators
```

`npm test` runs exactly:

```bash
firebase emulators:exec --config ../firebase.json --only firestore,storage --project demo-kin-care-connect "npm run test:run"
```

Other scripts:

```bash
npm run typecheck         # tsc --noEmit only, no emulators
npm run test:firestore    # Firestore emulator only (skips the 26 Storage tests)
```

Emulator ports come from `firebase.json` (Firestore 8080, Storage 9199). The first run downloads the emulator binaries to `~/.cache/firebase/emulators`.

What is covered: unauthenticated access everywhere; user documents (consent record, field whitelist, immutable fields, legacy phone removal); families, memberships and invitations by role (admin, caregiver, viewer, elderly) including declined/pending/removed members and cross-family access; care profiles (sharing off by default, who may change consent, revocation); activity records, medical summaries, reminders and their history, emergency contacts, recordings metadata, scam checks, notifications; server-only collections; Storage profile photos and consented recordings. See `docs/TESTING_AND_QA.md`.

**Deploying `storage.rules`:** the recordings rule reads the Firestore consent record (`firestore.get`). The first `firebase deploy --only storage` asks to grant the Storage service agent read access to Firestore; accept it, or uploads of recordings will be refused.

## 7. CI/CD (GitHub Actions)

`.github/workflows/ci.yml` runs on every push/PR: `flutter analyze`, `flutter test`, debug APK artifact, functions build + tests, and the security-rules tests against the Firestore and Storage emulators (Java 21, Node 22). On `main` it also builds a **signed** APK + AAB if these repository secrets exist:

| Secret | Value |
|---|---|
| `FIREBASE_OPTIONS_DART` | `base64 -w0 lib/firebase_options.dart` |
| `ANDROID_KEYSTORE_BASE64` | `base64 -w0 kincare-upload.jks` |
| `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_PASSWORD`, `ANDROID_KEY_ALIAS` | from step 6 |

## 8. Verification done so far

- `flutter analyze`: **no issues** (Flutter 3.47.9).
- `flutter test`: **115 tests passing** (unit + widget, incl. contrast ratios, legal-text sync, data minimisation, consent and age-gate flows, tap-target/label/contrast guidelines, keyboard traversal).
- `functions`: TypeScript strict build passes; **28 backend unit tests passing** (`npm test`, incl. deletion coverage and no-analytics/no-email checks).
- `rules-tests`: **299 Firestore rules tests passing** in the Firestore emulator (5 Nov 2026). The **26 Storage rules tests compile but were not executed** in the build sandbox (the Storage emulator's rules-runtime download was blocked there); CI and your machine run them.
- **Not yet verified, needs your environment:** an Android APK build (needs the Android SDK/JDK), the Storage rules tests (run `cd rules-tests && npm test`), live account deletion/export against Firebase, TalkBack/Switch Access/D-pad on a real device, and live calls to Firebase and the AI/speech providers (need your project and keys). See `docs/TESTING_AND_QA.md` for the checklist to run before release.

## 9. Important limitations (by design)

- **Routine Monitor** records only while the app is open and monitoring is switched on. It does **not** detect falls or emergencies and never sends emergency alerts. Background monitoring is deliberately not implemented (see the brief's "future implementation" notes).
- **Medical Translator** explains; it never diagnoses, prescribes, or advises on doses. Server-side guardrails remove such text if a model produces it.
- **Scam Shield** is advisory and can be wrong in both directions. The on-device check sends nothing anywhere; the AI check requires explicit consent.
- **Phone OTP sign-in is not implemented** (email/password only), so the app does not claim it.
- The legal texts (in the app and in `docs/legal/`) are **templates — need lawyer review** for your jurisdiction (e.g. India's DPDP Act 2023 and Rules).
- The app is for **adults (18+)**. Children's data is not knowingly collected; verifiable parental consent (DPDP s.9) would be needed before that could change.

## Release-signing safety
`flutter build apk --release` / `appbundle --release` now **fails** unless `android/key.properties` exists, so a debug-signed release can't be shipped by mistake. For a throwaway local test only: `ALLOW_DEBUG_SIGNED_RELEASE=true flutter build apk --release` (never distribute it).

## Evidence for auditors
- CI (`.github/workflows/ci.yml`) reruns `flutter analyze`, `flutter test`, the functions build and `npm test` on every push. Link the GitHub Actions run logs as independent evidence.
- Still owner-side: debug APK on a physical device plus `adb logcat`, Firebase Emulator rules tests, live export/deletion on a test Firebase project, and TalkBack/Switch Access/D-pad checks. See docs/TESTING_AND_QA.md.
