# NutriClinic – Dr. Yousfi Ibtissam (v4.1)

**Beginner? Read `GUIDE_AR.md` first** (Netlify hosting supported: just drag `firebase/hosting/public` to app.netlify.com/drop).

Kotlin + Jetpack Compose + Firebase (Auth, Firestore, Hosting, Cloud Functions, Cloud Messaging, App Check).

Doctor creates a questionnaire -> patient opens it on his own phone (link / QR / code) ->
answers reach the doctor in real time **and as a push notification even when the app is closed**.

## Project layout
- `app/` – Android app (package `nutriclinic.yousfi.aistudio.com`)
- `firebase/firestore.rules` – security rules
- `firebase/functions/` – Cloud Function that sends the push notification
- `firebase/hosting/public/` – web page behind the shared link + `assetlinks.json`
- `firebase/firebase.json` – deploy configuration

## Setup – follow in order

### 1. Create the Firebase project
1. https://console.firebase.google.com -> Add project (name e.g. `nutriclinic-yousfi`). Analytics: optional.
2. Project settings -> Your apps -> Android -> package name `nutriclinic.yousfi.aistudio.com`.
3. Add your SHA-1 (see step 6) -> Register -> download `google-services.json`.
4. Replace `app/google-services.json` with the downloaded file.

### 2. Authentication
1. Build -> Authentication -> Get started -> Sign-in method: enable **Email/Password** and **Anonymous**.
2. Users tab -> Add user: the doctor's email + a strong password.
3. Copy that user's **UID** (column "User UID").
4. Settings tab -> User actions: untick **Enable create (sign-up)** so nobody else can register.

### 3. Firestore
1. Build -> Firestore Database -> Create database -> Production mode -> region (e.g. `eur3`/`europe-west1`).
2. Open `firebase/firestore.rules` and replace `REPLACE_WITH_DOCTOR_UID` with the UID from step 2.3.

### 4. Firebase CLI
```
npm install -g firebase-tools
firebase login
cd firebase
firebase use --add          # choose your project
```

### 5. Cloud Functions (push notifications) – requires the Blaze plan
Functions need the pay-as-you-go plan (Project settings -> Usage and billing -> Modify plan). A clinic's volume stays inside the free monthly quota; set a budget alert anyway.
```
cd functions && npm install && cd ..
firebase deploy --only functions
```
If the CLI asks to enable APIs, answer yes. The function region is `europe-west1` (change it in `functions/index.js` if you want).

### 6. Hosting + Rules + App Links
1. In `app/build.gradle.kts` set `webHost` to `<project-id>.web.app` (exactly your Hosting domain).
2. Get your signing fingerprints: `./gradlew signingReport` (Android Studio: Gradle -> app -> Tasks -> android -> signingReport).
   * Add the **debug SHA-1** in Firebase (step 1.3) for testing.
   * For release add the **release SHA-1/SHA-256** too (and the Play App Signing SHA-256 once on Google Play).
3. Put the SHA-256 in `firebase/hosting/public/.well-known/assetlinks.json` (several fingerprints allowed).
4. Deploy:
```
firebase deploy --only firestore:rules,firestore:indexes,hosting
```
5. Test: `https://<project-id>.web.app/q/DRY-ABCD-2345` must show the fallback page.

### 7. App Check (protects your backend from other apps/scripts)
1. Firebase console -> Build -> App Check -> Apps -> Android -> register with **Play Integrity** (needs the app on Google Play, even internal testing) and add the release SHA-256.
2. Run a **debug** build once, find the line `Enter this debug secret into the allow list` in Logcat (tag `DebugAppCheckProvider`), then App Check -> app menu -> Manage debug tokens -> add it.
3. Check that requests are verified (App Check -> APIs) for a few days, **then** click **Enforce** on Cloud Firestore.
   (Enforcing before step 7.2 locks your own debug builds out.)

### 8. Build & run
Android Studio (Ladybug+), JDK 17 -> open `NutriClinic/` -> Sync -> Run.
If it complains about the Gradle wrapper jar, run `gradle wrapper --gradle-version 8.11.1` once, or just use Android Studio's bundled Gradle.

### 9. First test
1. Doctor phone: "I'm the doctor" -> sign in -> allow notifications.
2. Create a questionnaire -> share the QR / link.
3. Patient phone (app installed): scan the QR (or enter the code) -> fill -> send.
4. Doctor phone (even with the app closed): notification appears -> tap -> evaluation screen opens.

## Notes
* Patients need the app installed; send the Play Store/APK link with the first message.
* Data is personal medical data: keep rules deployed, strong doctor password, enable App Check enforcement.
* Fonts: the app uses the system font (Arabic renders natively). To use Cairo put `cairo.ttf` in `res/font/` and edit `ui/theme/Type.kt`.
