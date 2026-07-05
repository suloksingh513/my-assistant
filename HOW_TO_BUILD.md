# How to Build the Private AI Assistant APK

## Prerequisites

- **Android Studio** (latest stable) — https://developer.android.com/studio  
  OR **Command-line tools** with Java 17+ and Android SDK installed.
- Internet connection (Gradle downloads dependencies on first build).

---

## Project Structure

```
PrivateAIAssistant/
├── AndroidManifest.xml               ← top-level manifest (copy into app/src/main/)
├── build.gradle                      ← project-level Gradle file
├── settings.gradle                   ← Gradle settings
├── HOW_TO_BUILD.md                   ← this file
├── app/
│   ├── build.gradle                  ← app-level Gradle file
│   └── proguard-rules.pro
└── com/privateai/assistant/
    ├── ConfigActivity.java
    ├── AssistantService.java
    └── BootReceiver.java
```

---

## Step 1 — Set up the Android Studio project

1. Open Android Studio → **New Project → No Activity**.
2. Set package name: `com.privateai.assistant`
3. Set minimum SDK: **API 26 (Android 8.0)**
4. Click **Finish**.

---

## Step 2 — Replace generated files with the provided source

1. Copy `AndroidManifest.xml` to:  
   `app/src/main/AndroidManifest.xml`  
   (overwrite the generated one).

2. Copy the three Java files into:  
   `app/src/main/java/com/privateai/assistant/`

3. Replace `app/build.gradle` with the provided `app/build.gradle`.

4. Replace `build.gradle` (project root) with the provided `build.gradle`.

5. Replace `settings.gradle` with the provided `settings.gradle`.

---

## Step 3 — Add your license key

In `ConfigActivity.java`, find:

```java
private static final String LICENSE = "YOUR_LICENSE_KEY_HERE";
```

Replace `YOUR_LICENSE_KEY_HERE` with your actual KeyAuth license key.

---

## Step 4 — (Recommended) Secure your OpenAI key

The OpenAI API key is currently hardcoded in `AssistantService.java`.  
**Anyone who decompiles the APK can extract it.**

**Safer alternative:** Remove the key from the source file and fetch it at runtime
from a private server you control (e.g., a simple HTTPS endpoint that returns the key
after verifying the device's KeyAuth token). This keeps the key out of the APK entirely.

---

## Step 5 — Build

### In Android Studio
- Click **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
- The APK appears in `app/build/outputs/apk/debug/` or `release/`.

### Command line
```bash
./gradlew assembleRelease
```
Output: `app/build/outputs/apk/release/app-release.apk`

---

## Step 6 — Sign for distribution (release builds)

For sideloading on your own device, the debug APK works fine.  
For wider distribution, create a keystore and configure `signingConfigs` in `app/build.gradle`.

---

## Step 7 — Install on the tablet

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

Or transfer the APK file to the tablet and open it via a file manager (enable
**Install from unknown sources** in Android settings first).

---

## Permissions the user must grant on first launch

| Permission | Why |
|---|---|
| `RECORD_AUDIO` | Continuous speech recognition |
| `READ_PHONE_STATE` | Hardware ID for KeyAuth |
| Battery optimization exemption | Keeps the service alive indefinitely |

The app will request `RECORD_AUDIO` and `READ_PHONE_STATE` on first launch.
The battery optimization dialog opens automatically.

---

## Behaviour after install

1. Launch the app — it will appear briefly in the launcher, then immediately become invisible.
2. KeyAuth verifies your license, then `AssistantService` starts.
3. The service keeps running invisibly in the background with a minimal system notification.
4. After reboot, `BootReceiver` restarts the service automatically.
5. Say **"Suno Assistant"** followed by your question to activate GPT-4o.
6. Say **"Bhej do"** to trigger the action intent (copies reply + fires `ACTION_SEND`).
