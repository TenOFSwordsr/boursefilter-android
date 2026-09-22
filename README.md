# BourseFilter Android (Etemad Bourse WebView Shell)

A minimal native Android wrapper for the "Etemad Bourse" (اعتماد بورس) web dashboard. The entire
app is one full-screen `WebView` pointed at the TSETMC filter server plus a small amount of glue:
in-app navigation, swipe-to-refresh, and handing Excel exports to the system `DownloadManager`.
The trading/filtering logic lives server-side, not here - this is a distribution shell so the
dashboard installs as an app icon. Persian-language UI (`android:label="اعتماد بورس"`).

**Suggested repo name:** `boursefilter-android`
**Stack:** Java (Android), Gradle KTS, AGP 8.5.2, AndroidX AppCompat + SwipeRefreshLayout
**Status:** finished (single-activity shell)
**Last modified:** 2026-09-02

## What it does

- `MainActivity` loads `DEFAULT_URL` (`<prod-host>:8080`) into a `WebView`, overriding
  `shouldOverrideUrlLoading` so every link stays inside the app instead of opening a browser.
- Enables JavaScript, DOM storage and the WebView database - the dashboard uses `localStorage` for
  the basket/portfolio and settings state.
- Catches download events (the dashboard's `.xlsx` export) and forwards them to `DownloadManager`,
  saving into the phone's `Downloads` folder with a MIME fallback to the Excel type.
- `SwipeRefreshLayout` gives pull-to-refresh; back button walks WebView history before exiting.
- Manifest declares `INTERNET` and `usesCleartextTraffic="true"` so plain-HTTP access to the
  server works on Android 9+ (the server is on the local network / a LAN-facing host).

## Layout

```
settings.gradle.kts            root project "BourseFilter", includes :app
build.gradle.kts               AGP version pin
app/build.gradle.kts           com.etemad.bourse, minSdk 24 / target+compile 35, deps
app/src/main/AndroidManifest.xml   permissions, single launcher activity
app/src/main/java/com/etemad/bourse/MainActivity.java   the WebView shell
app/src/main/res/layout/activity_main.xml               WebView + SwipeRefreshLayout
app/src/main/res/...            launcher icons, styles
```

## Running it

No Gradle wrapper is committed (`gradle/wrapper/` is empty, there is no `gradlew`/`gradlew.bat`), so
open the folder in Android Studio or invoke a locally installed Gradle:

```bash
gradle :app:assembleDebug     # output: app/build/outputs/apk/debug/app-debug.apk
```

A debug APK from a previous build is present at that path. The device must be able to reach
`<prod-host>:8080` (or edit `DEFAULT_URL`) for the dashboard to load.

## Notes

- **Hard-coded plaintext server.** `DEFAULT_URL` is an `http://` IP address and the manifest allows
  cleartext traffic. For any public release, move to a TLS host and drop `usesCleartextTraffic`.
- This is the Android half of the same product as the EtemadBourse Python/Go server; the delivered
  source drop ships near-identical files under `Android_App/` (its `MainActivity.java` is currently
  byte-identical to this one). Keep the two in sync or treat one as canonical.
- `local.properties` (SDK path) and `app/build/` are machine-specific build output - gitignore them;
  neither belongs in a published repo.
- Comments and identifiers mix Persian and English intentionally; the build reads them fine as UTF-8.
