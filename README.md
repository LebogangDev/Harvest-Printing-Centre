# Harvest Printing Centre - Android app (Task 2)

Kotlin app (XML views, Navigation, ViewBinding, MVVM) built by modifying the existing
Harvest app. Package `com.harvestprintingcentre.app` is kept so it still matches the
Firebase project `harvest-printing-centre`.

Put this folder in the repo's `app/` folder (open `app/` in Android Studio).

## 1. Setup checklist

1. **google-services.json** - Firebase console > Project settings > Your apps > Android
   (`com.harvestprintingcentre.app`). Keep it in `app/app/` (it is in `.gitignore`).
2. **Sign-in methods** - Authentication > Sign-in method: enable **Email/Password** and **Google**.
   The Google **Web client ID** goes in `gradle.properties` -> `harvest.webClientId=`.
3. **Register this build's SHA-1 (required for Google sign-in).** See section 2.
4. **API address** - `harvest.apiBaseUrl` is already set to Given's live server
   (`https://harvest-printing-api.onrender.com/`).

Optional: Authentication > Settings > User actions > turn off "Email enumeration protection"
to get *separate* "wrong password" and "unknown email" messages.

## 2. Google sign-in: "can't find a Google account" fix

That message is not about the phone. Google only lists accounts to apps whose **signing key
(SHA-1)** is registered in Firebase, and the key belongs to your build (your PC's debug
keystore), so every phone fails the same way. In `google-services.json`, your app must contain
an `oauth_client` with `"client_type": 1` and a `certificate_hash`. If it only has `client_type: 3`,
the SHA-1 is missing.

1. Run the app and tap "Sign in with Google" once. In a debug build the error on screen shows
   your SHA-1 (it is also in Logcat, filter tag `GoogleSignIn`). Or run `./gradlew signingReport`
   and copy the `debug` SHA-1.
2. Firebase console > Project settings > Your apps > `com.harvestprintingcentre.app` >
   **Add fingerprint** > paste the SHA-1 > Save.
3. Download `google-services.json` again, replace `app/app/google-services.json`, then
   **Sync** and **Run** again. Every teammate who builds the app needs their own SHA-1 added too.

The app tries the account chooser first and falls back to the classic "Sign in with Google" flow.

## 3. Given's API (https://harvest-printing-api.onrender.com)

| Call | Auth | Purpose |
|---|---|---|
| `GET /services` | public | services (`id, name, category, description, price, priceLabel, imageFile`) |
| `GET /orders` | Bearer token | the signed-in user's orders |
| `POST /orders` | Bearer token | body `{"items":[{"serviceId","quantity"}]}`; server prices the order |

The server is on Render's free tier: after idle time the first request can take up to a minute
(the app waits and says so). Open the base URL in a browser before a demo to wake it.
Services with no price (Programmes, Flags, Photo Canvas) are marked "Quote needed".
If the JSON changes, only `data/remote/dto/Dtos.kt` needs editing.

## 4. Service photos

The photos are bundled in `app/src/main/res/drawable-nodpi/svc_*.jpg` and chosen by the API's
`imageFile` (falling back to the service id) in `util/ServiceImageKey.kt` and
`ui/common/ServiceImages.kt`. Tap a photo to view it full size. To add a photo for a new
service: add the file, add its key to both files.

## 5. Task 2 plan -> where it is

| Step | Done in |
|---|---|
| A Project, bottom nav Home/Orders/Services/Profile | `MainActivity`, `nav_graph.xml`, `bottom_nav_menu.xml` |
| B Firebase | `build.gradle.kts`, `google-services.json` |
| C Register / login, errors, stay signed in | `AuthRepository`, `AuthErrors`, `LoginFragment`, `RegisterFragment`; session restored via `FirebaseAuth.currentUser` in `MainActivity` |
| D Google SSO | `GoogleSignInHelper` (Credential Manager) + `AuthRepository.loginWithGoogle`; sign-out returns to Login |
| E Retrofit API | `ApiService`, `ApiClient`, `ApiResult.safeApiCall`, repositories; unreachable server -> message |
| F Services + Orders screens | `ui/services`, `ui/orders` (New order / My orders tabs) |
| G Logging, comments, JUnit | `Log.d/Log.e` (tags per class), KDoc comments, `app/src/test` |
| H Real phone | see section 5 |

Settings and Help under Profile: Settings is the original placeholder screen and Help opens
WhatsApp support chat, so Given can replace them without touching anything else.

## 6. What changed from the old app

- Login accepted any text and signed in as "John Doe" -> real Firebase Auth + Google.
- Orders went to WhatsApp / Firestore -> saved through the API; orders list comes from the API.
- Static product lists -> services from the API.
- Removed: Quotes screen (not in navigation), favourites, and the placeholder "3 saved designs /
  2 addresses / payments" rows (they were not connected to any data).
- Fixed: home header showed the default Android robot instead of the logo; notification button
  now asks for the Android 13+ permission; layouts respect system bars on Android 15;
  background images shrunk (1.4 MB PNG -> ~60 KB WebP).

## 7. Run on a real phone

Phone: Settings > About phone > tap Build number 7 times > Developer options > USB debugging on.
Plug in, accept the prompt, pick the phone in Android Studio's device list, press Run.

## 8. Tests

`./gradlew test` (or right-click `app/src/test` > Run). Covers validation, price parsing, order
totals, JSON parsing, the whole API path against a mock server (token header, 401/500, bad JSON,
unreachable server, empty 201), and Firebase error mapping.

## 9. Say this in the report / video

- Firebase Authentication never stores plain-text passwords: the app sends the password over
  HTTPS and Firebase stores only a salted hash.
- Google sign-in uses Credential Manager to get a Google ID token, exchanged for a Firebase
  session with `GoogleAuthProvider.getCredential` + `signInWithCredential`.
- API calls carry the Firebase ID token as a Bearer header so Given's API can verify the user.
- Failures (offline, 401, 500, bad JSON) become on-screen messages with a retry button.

## 10. References (Harvard - add your access date)

- Google (2025) *Sign in your user with Credential Manager*. https://developer.android.com/identity/sign-in/credential-manager-siwg
- Google (2025) *Authenticate with Firebase using password-based accounts on Android*. https://firebase.google.com/docs/auth/android/password-auth
- Google (2025) *Authenticate with Firebase using Google Sign-In on Android*. https://firebase.google.com/docs/auth/android/google-signin
- Square (2025) *Retrofit*. https://square.github.io/retrofit/
- Code adapted from the team's original Harvest app: `NotificationHelper`, banner slider, WhatsApp helper, layouts, colours.
