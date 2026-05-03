# Android Emulator And ADB Fixes

Use this file as Android emulator, ADB, networking, and Laravel/NativePHP mobile troubleshooting training/reference data.

## Quick Commands

List emulators:

```bash
emulator -list-avds
```

Start emulator:

```bash
emulator -avd <name>
emulator @<name>
```

Start with software graphics:

```bash
emulator -avd <name> -gpu software
emulator -avd <name> -gpu swiftshader
```

Legacy GPU fallback:

```bash
emulator -avd <name> -gpu swiftshader_indirect
```

Cold boot:

```bash
emulator -avd <name> -no-snapshot-load
```

Wipe data:

```bash
emulator -avd <name> -wipe-data
```

Restart ADB:

```bash
adb kill-server
adb start-server
adb devices
```

## Environment Variables

Windows PowerShell temporary session:

```powershell
$env:ANDROID_HOME="C:\Users\<user>\AppData\Local\Android\Sdk"
$env:ANDROID_SDK_ROOT="C:\Users\<user>\AppData\Local\Android\Sdk"
$env:Path += ";$env:ANDROID_HOME\platform-tools"
$env:Path += ";$env:ANDROID_HOME\emulator"
```

Permanent variables should be set in Windows Environment Variables:

```text
ANDROID_HOME=C:\Users\<user>\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT=C:\Users\<user>\AppData\Local\Android\Sdk
Path += %ANDROID_HOME%\platform-tools
Path += %ANDROID_HOME%\emulator
```

Verify:

```bash
adb version
emulator -version
emulator -list-avds
java -version
```

## Common Problems

Black screen:

```text
Likely GPU/rendering issue.
Try cold boot.
Try -gpu software.
Try -gpu swiftshader.
Update Android Emulator from SDK Manager.
```

Device offline:

```text
Restart ADB.
Unplug/replug physical device.
Revoke USB debugging authorizations on device.
Accept RSA prompt on device.
Cold boot emulator.
```

Laravel not accessible:

```text
Use http://10.0.2.2:8000 from Android emulator.
Run Laravel with --host=0.0.0.0.
Check firewall.
Check actual port.
Clear Laravel config cache.
```

NativePHP app blank:

```text
Open Laravel URL in desktop browser first.
Check storage/logs/laravel.log.
Check adb logcat.
Check APP_URL.
Check Vite dev server or production build.
```

## Emulator Networking

Important mapping:

```text
Emulator 127.0.0.1 = Android emulator itself
Emulator 10.0.2.2 = host computer localhost
```

Laravel on host:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

URL from emulator:

```text
http://10.0.2.2:8000
```

URL from host:

```text
http://127.0.0.1:8000
```

Physical Android device on same Wi-Fi:

```text
http://<computer-lan-ip>:8000
```

Physical Android device over USB:

```bash
adb reverse tcp:8000 tcp:8000
```

Then on device:

```text
http://127.0.0.1:8000
```

## ADB Troubleshooting

List devices:

```bash
adb devices
adb devices -l
```

Restart server:

```bash
adb kill-server
adb start-server
adb devices
```

Kill emulator:

```bash
adb -s emulator-5554 emu kill
```

Install APK:

```bash
adb install app-debug.apk
adb install -r app-debug.apk
```

Uninstall package:

```bash
adb uninstall com.example.app
```

Clear app data:

```bash
adb shell pm clear com.example.app
```

Open logs:

```bash
adb logcat
```

Filter logs on Windows:

```powershell
adb logcat | findstr NativePHP
adb logcat | findstr Laravel
adb logcat | findstr AndroidRuntime
```

## Emulator Performance

Improve performance:

- Use x86_64 system images when available.
- Enable hardware virtualization in BIOS/UEFI.
- Use recent Android Emulator and platform tools.
- Allocate enough RAM to AVD.
- Close unused heavy apps.
- Prefer `-gpu auto` or `-gpu host` when stable.
- Use `-gpu software` when host GPU causes black screens or crashes.

Useful options:

```bash
emulator -avd <name> -memory 2048
emulator -avd <name> -no-snapshot-load
emulator -avd <name> -no-snapshot-save
emulator -avd <name> -no-snapshot
```

## Android Studio Fixes

SDK Manager checklist:

```text
Android SDK Platform
Android SDK Build-Tools
Android SDK Platform-Tools
Android Emulator
Google APIs system image
```

AVD Manager checklist:

```text
Cold Boot Now
Wipe Data
Edit AVD -> Graphics -> Software
Create a fresh AVD if old one is corrupted
```

Gradle/build issues:

```text
Check JDK version.
Check Android Gradle Plugin compatibility.
Check SDK licenses.
Install missing SDK packages.
```

Accept SDK licenses:

```bash
sdkmanager --licenses
```

## Laravel And NativePHP Emulator Checklist

Laravel server:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

Clear Laravel caches:

```bash
php artisan optimize:clear
```

NativePHP run:

```bash
php artisan native:run android
php artisan native:run android --watch
```

Debug checklist:

- Is Laravel working in the desktop browser?
- Is the emulator running?
- Does `adb devices` show `device` and not `offline`?
- Is `APP_URL` correct?
- Is Windows firewall allowing the port?
- Are assets built or is Vite running?
- Does `storage/logs/laravel.log` show an exception?
- Does `adb logcat` show an Android crash?

## Error Patterns

`ERR_CONNECTION_REFUSED`:

```text
Server is not running, wrong host, wrong port, firewall, or using localhost from emulator.
```

`offline` in `adb devices`:

```text
Restart ADB, unlock device, accept debugging prompt, cold boot emulator.
```

Black screen after launch:

```text
GPU issue, app crash, missing assets, wrong URL, or Laravel exception.
```

Build tool not found:

```text
ANDROID_HOME/ANDROID_SDK_ROOT/Path issue or missing SDK package.
```

`INSTALL_FAILED_VERSION_DOWNGRADE`:

```bash
adb uninstall com.example.app
adb install app-debug.apk
```

`INSTALL_FAILED_UPDATE_INCOMPATIBLE`:

```bash
adb uninstall com.example.app
adb install app-debug.apk
```
