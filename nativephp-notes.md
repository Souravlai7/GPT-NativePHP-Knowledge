# NativePHP Notes

Use this file as NativePHP for Mobile and Laravel native app training/reference data. NativePHP lets a Laravel app run in a native shell for platforms such as Android and iOS. Confirm exact package/version behavior against the project because NativePHP evolves quickly.

## Install NativePHP Mobile

Install in a Laravel app:

```bash
composer require nativephp/mobile
php artisan native:install
php artisan native:run
```

After install, NativePHP may provide a helper command:

```bash
php native run
./native run
```

Common platform arguments:

```bash
php artisan native:run android
php artisan native:run ios
php artisan native:run android --watch
php artisan native:run android --build=debug
php artisan native:run android --build=release
php artisan native:run android --start-url=/dashboard
```

Current command pattern:

```text
php artisan native:run {os?} {udid?}
os can be android/a or ios/i
udid targets a specific device/emulator
```

## Native Install Options

Useful install options:

```bash
php artisan native:install
php artisan native:install android
php artisan native:install ios
php artisan native:install both
php artisan native:install --force
php artisan native:install --fresh
php artisan native:install --with-icu
php artisan native:install --without-icu
php artisan native:install --skip-php
```

Notes:

- `--force` or `--fresh` overwrites generated native files.
- Treat generated NativePHP platform folders as rebuildable unless the project intentionally customizes them.
- If a generated folder is documented as ephemeral for the installed NativePHP version, avoid hand-editing it.
- Re-run install after NativePHP upgrades when the generated native project must be rebuilt.

## Development Workflow

Recommended workflow:

```bash
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm run dev
php artisan serve
php artisan native:run android --watch
```

Before running native, test the Laravel app in a normal browser:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

Then open:

```text
http://127.0.0.1:8000
```

If it fails in browser, fix Laravel first before debugging NativePHP.

## Android Requirements

Install:

```text
Android Studio
Android SDK
Android SDK Platform Tools
Android Emulator
Android SDK Build Tools
JDK required by Android Gradle plugin
```

Environment variables on Windows:

```powershell
$env:ANDROID_HOME="C:\Users\<user>\AppData\Local\Android\Sdk"
$env:ANDROID_SDK_ROOT="C:\Users\<user>\AppData\Local\Android\Sdk"
$env:Path += ";$env:ANDROID_HOME\platform-tools"
$env:Path += ";$env:ANDROID_HOME\emulator"
```

Check tools:

```bash
adb version
emulator -version
emulator -list-avds
java -version
```

## Running Android Emulator

List AVDs:

```bash
emulator -list-avds
```

Start emulator:

```bash
emulator -avd Pixel_8_API_35
emulator @Pixel_8_API_35
```

Cold boot:

```bash
emulator -avd Pixel_8_API_35 -no-snapshot-load
```

Wipe emulator data:

```bash
emulator -avd Pixel_8_API_35 -wipe-data
```

Software graphics fallback:

```bash
emulator -avd Pixel_8_API_35 -gpu software
emulator -avd Pixel_8_API_35 -gpu swiftshader
```

Legacy fallback:

```bash
emulator -avd Pixel_8_API_35 -gpu swiftshader_indirect
```

Note: recent Android Emulator docs mark `swiftshader_indirect` as deprecated. Prefer `-gpu software` or `-gpu swiftshader` first, then use legacy flags only when working with older emulator versions.

## ADB Commands

Check devices:

```bash
adb devices
```

Restart ADB:

```bash
adb kill-server
adb start-server
adb devices
```

Install APK:

```bash
adb install app-debug.apk
adb install -r app-debug.apk
```

Uninstall app:

```bash
adb uninstall com.example.app
```

View logs:

```bash
adb logcat
adb logcat | findstr NativePHP
adb logcat | findstr Laravel
```

Open shell:

```bash
adb shell
```

Reverse port for physical Android device:

```bash
adb reverse tcp:8000 tcp:8000
```

## Emulator Networking

Important addresses:

```text
127.0.0.1 inside Android emulator means the emulator itself.
10.0.2.2 from Android emulator points to host machine localhost.
```

Laravel running on host:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

From Android emulator:

```text
http://10.0.2.2:8000
```

From physical Android device on same Wi-Fi:

```text
http://<computer-lan-ip>:8000
```

Find computer LAN IP on Windows:

```powershell
ipconfig
```

For physical USB device, use:

```bash
adb reverse tcp:8000 tcp:8000
```

Then device can access:

```text
http://127.0.0.1:8000
```

## NativePHP App Not Loading

Checklist:

- Confirm Laravel runs in browser first.
- Confirm `.env` has correct `APP_URL`.
- Run `php artisan optimize:clear`.
- Confirm Android emulator has network access.
- Use `10.0.2.2` for emulator-to-host access.
- Use `adb reverse` for physical USB device.
- Check Windows firewall for blocked PHP/server port.
- Confirm the dev server port matches NativePHP start URL.
- Check `storage/logs/laravel.log`.
- Check `adb logcat`.

Commands:

```bash
php artisan optimize:clear
php artisan serve --host=0.0.0.0 --port=8000
adb devices
adb logcat
```

## Laravel Config For Native Context

Common `.env` development values:

```env
APP_ENV=local
APP_DEBUG=true
APP_URL=http://10.0.2.2:8000
SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=sync
```

When using a physical device:

```env
APP_URL=http://192.168.1.50:8000
```

After changing `.env`:

```bash
php artisan config:clear
php artisan optimize:clear
```

## Database In Native Apps

Development options:

- Use host MySQL from emulator through host IP if network allows.
- Use API calls to a remote backend.
- Use SQLite for local app storage if supported by the app design.

For emulator to host MySQL:

```env
DB_HOST=10.0.2.2
DB_PORT=3306
```

For physical device to host MySQL:

```env
DB_HOST=<computer-lan-ip>
DB_PORT=3306
```

Security:

- Do not ship production database credentials inside a mobile app.
- Prefer API authentication to direct database access for deployed mobile apps.
- Treat anything bundled in an APK/IPA as discoverable by users.

## Auth And Sessions

Native/mobile considerations:

- Cookie sessions can work for embedded web views but need careful domain, secure, and SameSite settings.
- Token auth is often simpler for API-style mobile communication.
- Laravel Sanctum can be used for SPA/API token patterns depending on architecture.
- Avoid relying on `localhost` cookies in emulator; use the correct host address.

Common session `.env` values for local development:

```env
SESSION_DRIVER=file
SESSION_SECURE_COOKIE=false
```

## Assets

For Vite development:

```bash
npm run dev
```

For production/native package builds:

```bash
npm run build
```

If CSS/JS does not update:

```bash
php artisan view:clear
npm run dev
```

Check:

- Vite dev server is running.
- Device can reach the Vite host/port if using hot reload.
- Production build files exist in `public/build`.

## Queues In Native Apps

For simple local native development:

```env
QUEUE_CONNECTION=sync
```

For server-backed apps:

```env
QUEUE_CONNECTION=database
```

Run worker:

```bash
php artisan queue:work
```

Mobile app guidance:

- Do not expect a mobile foreground app to behave like a permanent queue worker.
- Push slow or reliable background work to a backend server when possible.
- Use native/platform background APIs only when the NativePHP version and plugins support the needed behavior.

## Permissions And Plugins

General guidance:

- Register NativePHP plugins when required.
- Rebuild after plugin or native permission changes.
- Android permissions may require manifest changes through supported NativePHP configuration/plugin mechanisms.
- iOS permissions may require Info.plist descriptions through supported configuration/plugin mechanisms.

Command noted in NativePHP docs:

```bash
php artisan native:plugin:register
```

## Build Types

Debug build:

```bash
php artisan native:run android --build=debug
```

Release build:

```bash
php artisan native:run android --build=release
```

Bundle build:

```bash
php artisan native:run android --build=bundle
```

Release checklist:

- Set production `.env` values.
- Disable debug mode.
- Build frontend assets.
- Clear and cache Laravel config where appropriate.
- Confirm API URLs point to production.
- Do not embed local-only URLs like `10.0.2.2`.
- Test on a real device.

## Common NativePHP Issues

Emulator not connecting:

```text
Use 10.0.2.2 instead of localhost.
Run php artisan serve --host=0.0.0.0 --port=8000.
Check firewall.
```

ADB offline:

```bash
adb kill-server
adb start-server
adb devices
```

App not loading:

```text
Check Laravel logs.
Check adb logcat.
Check APP_URL.
Check port.
Check Vite/build assets.
```

Build fails:

```text
Check Java version.
Check Android SDK path.
Check Gradle output.
Run native install again after package upgrades.
Clear Laravel caches.
```

Blank screen:

```text
Open the same URL in browser.
Check JavaScript console if available.
Check built assets.
Check adb logcat.
Try emulator software GPU.
```

## NativePHP Training Rules

- NativePHP apps are still Laravel apps; debug Laravel first.
- Browser success should come before native debugging.
- Emulator `localhost` is not host `localhost`.
- Use `10.0.2.2` for Android emulator to reach host server.
- Use `adb reverse` for USB physical device when appropriate.
- Never ship local development URLs in production mobile builds.
- Do not ship database root credentials in mobile apps.
- Prefer backend APIs for production mobile apps.
- Rebuild native projects after changing plugins, permissions, or native config.
