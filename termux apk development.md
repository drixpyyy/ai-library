# SKILL: Build Android APKs in Termux
**Tested & working** — debugged and verified on-device.  
Use this as a baseplate for any Android app built entirely from a phone via Termux.

---

## Prerequisites (one-time setup)
Run with `RUN_SETUP=true` on a fresh Termux install.  
After that, set `RUN_SETUP=false` — setup takes ~15–30 min and only needs to run once.

### What gets installed
| Tool | Purpose |
|---|---|
| `openjdk-21` | Java compiler |
| `aapt2` | Android asset packager |
| `gradle 8.5` | Build system |
| Android SDK API 34 | Target platform |
| Build Tools 34.0.0 | dx, apksigner, zipalign |
| `python` + `pillow` | Icon generation |

---

## Script Configuration Variables
```bash
RUN_SETUP=false           # true = install SDK/Gradle/etc (first run only)
PROJECT_NAME="SysInfo"    # Used for folder name and APK filename
PACKAGE_NAME="com.example.sysinfo"  # Java package — must be unique per app
SDK_API_LEVEL="34"
BUILD_TOOLS_VER="34.0.0"
```

---

## Project Structure Generated
```
$HOME/ProjectName/
├── settings.gradle
├── build.gradle                    # Top-level (buildscript block)
├── debug.keystore                  # Auto-generated on first build
└── app/
    ├── build.gradle                # App-level (plugins, android{}, dependencies)
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/example/sysinfo/
        │   └── MainActivity.java
        └── res/
            ├── layout/activity_main.xml
            ├── values/strings.xml
            ├── values/themes.xml
            ├── values/styles.xml
            └── mipmap-{hdpi,mdpi,xhdpi,xxhdpi,xxxhdpi}/
                ├── ic_launcher.png
                └── ic_launcher_round.png
```

---

## Key Patterns & Gotchas

### gradle.properties (critical for Termux)
Always write these two lines — without them the build will fail:
```
android.aapt2FromMavenOverride=/data/data/com.termux/files/usr/bin/aapt2
android.useAndroidX=true
```

### Java version
Stick to `JavaVersion.VERSION_1_8` in `compileOptions`. Higher versions can cause issues in Termux's JVM environment.

### Kotlin stdlib conflict
If adding any library that pulls in Kotlin, exclude it explicitly:
```groovy
configurations.implementation {
    exclude group: 'org.jetbrains.kotlin', module: 'kotlin-stdlib-jdk8'
}
```

### Icon generation (Python/Pillow)
The script generates icons programmatically. Swap this block to customize:
```python
sizes = {"mipmap-mdpi":48,"mipmap-hdpi":72,"mipmap-xhdpi":96,
         "mipmap-xxhdpi":144,"mipmap-xxxhdpi":192}
# Change color '#2196F3' and text 'Sys' to customize
img = Image.new('RGB',(size,size),color='#2196F3')
```

### APK signing (debug key)
Auto-generates a debug keystore on first build. For distribution you'd replace with a proper release key, but for sideloading this works fine:
```bash
keytool -genkey -v -keystore debug.keystore -storepass android \
  -alias androiddebugkey -keypass android -keyalg RSA \
  -keysize 2048 -validity 10000 \
  -dname "CN=Android Debug,O=Android,C=US"
```

### Output location
Signed APK lands at:
```
$HOME/storage/shared/ProjectName.apk
```
This is your phone's shared storage — easy to install from a file manager.

---

## Dependencies Template
Paste into `app/build.gradle` under `dependencies {}`:
```groovy
// Core AndroidX
implementation 'androidx.appcompat:appcompat:1.6.1'
implementation 'androidx.core:core:1.12.0'
implementation 'com.google.android.material:material:1.11.0'

// If adding more libs, watch for Kotlin stdlib conflicts (see above)
```

---

## Common Permissions Reference
Add to `AndroidManifest.xml` inside `<manifest>`:
```xml
<!-- Sensors -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />

<!-- Device info -->
<uses-permission android:name="android.permission.BATTERY_STATS" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<!-- Network -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- Storage -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

---

## Build Commands
```bash
# Full build (run from project root)
gradle assembleDebug

# If build fails, clean first
gradle clean assembleDebug

# Check Gradle version
gradle --version

# Check Java
java -version
```

---

## Troubleshooting

| Error | Fix |
|---|---|
| `aapt2` not found | Check gradle.properties has the override path |
| `Could not resolve` dependency | Check internet; Termux may need `pkg install ca-certificates` |
| Kotlin stdlib conflict | Add the exclusion block shown above |
| `keytool` not found | Java not on PATH — re-export `JAVA_HOME` |
| Build succeeds but APK won't install | Enable "Install from unknown sources" in Android settings |
| `storage/shared` not found | Run `termux-setup-storage` first |

---

## Extending This Baseplate

### To add microphone access (the thing browsers can't do on mobile)
1. Add `RECORD_AUDIO` permission to manifest
2. Request it at runtime in `MainActivity.java`:
```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO)
        != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this,
        new String[]{Manifest.permission.RECORD_AUDIO}, 1);
}
```
3. Use `AudioRecord` class for raw audio capture — this is how you'd do real echolocation/sonar data collection natively.

### To add multiple screens
Add new `Activity` classes and declare them in `AndroidManifest.xml`:
```xml
<activity android:name=".SecondActivity" android:exported="false"/>
```
