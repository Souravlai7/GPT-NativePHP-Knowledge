# nativephp core


---

# Source: nativephp-notes.md

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


---

# Source: nativephp-auto-training.md

# NativePHP Auto Training And Response Playbook

Use this file to train GPT how to automatically respond when the user asks about NativePHP, Laravel mobile apps, Android emulator, NativePHP posts/forms, API calls, app blank screens, database access, builds, or deployment.

## Intent Detection

If user mentions any of these, treat the issue as NativePHP-related:

```text
nativephp
native php
php artisan native
native:run
native:install
native app
android app
ios app
apk
aab
emulator
adb
10.0.2.2
webview
mobile laravel
app blank
app not loading
native post
form submit in native
api call from native
```

## Default NativePHP Answer Pattern

For any NativePHP problem, GPT should answer in this order:

1. Identify the likely layer.
2. Confirm Laravel works in a normal browser.
3. Confirm device/emulator networking.
4. Check NativePHP command/build.
5. Check logs.
6. Give exact commands.
7. Mention production warnings if relevant.

Template:

```text
Most likely layer: Laravel app, NativePHP shell, Android emulator networking, or build/assets.

Try this:
1. Run Laravel in browser first.
2. Serve with `php artisan serve --host=0.0.0.0 --port=8000`.
3. From Android emulator use `http://10.0.2.2:8000`.
4. Clear Laravel cache with `php artisan optimize:clear`.
5. Run `php artisan native:run android --watch`.
6. Check `storage/logs/laravel.log` and `adb logcat`.
```

## If User Says NativePHP App Not Loading

Most likely causes:

- Laravel app has an exception.
- Native app cannot reach Laravel URL.
- Wrong `APP_URL`.
- Vite/build assets missing.
- Android emulator networking issue.
- Windows firewall blocks the port.

Response:

```bash
php artisan optimize:clear
php artisan serve --host=0.0.0.0 --port=8000
php artisan native:run android --watch
adb devices
adb logcat
```

Tell user:

```text
Open Laravel in desktop browser first. If browser fails, it is not a NativePHP problem yet. Fix Laravel first.
```

Android emulator URL:

```text
http://10.0.2.2:8000
```

Physical device URL:

```text
http://<computer-lan-ip>:8000
```

USB physical device option:

```bash
adb reverse tcp:8000 tcp:8000
```

## If User Says POST/Form Submit Not Working In NativePHP

Likely causes:

- Missing CSRF token.
- Session/cookie not persisting.
- Wrong action URL.
- Native webview cannot reach server.
- Validation error hidden.
- API route expects JSON but form sends normal POST.
- `APP_URL` or route URL points to localhost incorrectly.

Checklist:

```bash
php artisan route:list
php artisan optimize:clear
```

Blade form should include:

```blade
<form method="POST" action="{{ route('posts.store') }}">
    @csrf
    <input name="title" value="{{ old('title') }}">
    @error('title')
        <p>{{ $message }}</p>
    @enderror
    <button type="submit">Save</button>
</form>
```

For file upload:

```blade
<form method="POST" action="{{ route('files.store') }}" enctype="multipart/form-data">
    @csrf
    <input type="file" name="file">
    <button type="submit">Upload</button>
</form>
```

If using JSON API from NativePHP:

```js
await fetch('/api/posts', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ title: 'Hello' }),
});
```

Laravel API route:

```php
Route::post('/posts', [PostApiController::class, 'store']);
```

Controller:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:255'],
    ]);

    return response()->json([
        'data' => Post::create($validated),
    ], 201);
}
```

GPT should ask for this only if needed:

```text
Are you submitting a Blade form route or a JSON API route?
```

## If User Says API Call Fails In NativePHP

Likely causes:

- Wrong base URL.
- CORS issue if calling external browser-style API.
- Token missing/expired.
- Device offline.
- SSL certificate problem.
- API returns validation error.

Debug with:

```text
Check Network request status.
Check Laravel log.
Check API response body.
Check token/header.
Check emulator URL mapping.
```

Fetch pattern:

```js
const response = await fetch(`${baseUrl}/api/posts`, {
  headers: {
    'Accept': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
});

if (!response.ok) {
  const error = await response.text();
  console.error(error);
}
```

GPT should warn:

```text
Do not use `localhost` from Android emulator to reach the host Laravel server. Use `10.0.2.2`.
```

## If User Says NativePHP Database Not Working

First determine architecture:

```text
Is the app using local SQLite, host MySQL during development, or a remote backend API?
```

Development emulator to host MySQL:

```env
DB_HOST=10.0.2.2
DB_PORT=3306
```

Physical device to host MySQL:

```env
DB_HOST=<computer-lan-ip>
DB_PORT=3306
```

After `.env` changes:

```bash
php artisan config:clear
php artisan optimize:clear
```

Production warning:

```text
Do not ship production database credentials inside a mobile app. Use a backend API for production.
```

## If User Says Emulator Issue

Commands:

```bash
emulator -list-avds
emulator -avd <name> -gpu software
adb kill-server
adb start-server
adb devices
```

For black screen:

```bash
emulator -avd <name> -no-snapshot-load
emulator -avd <name> -gpu software
emulator -avd <name> -gpu swiftshader
```

Legacy fallback:

```bash
emulator -avd <name> -gpu swiftshader_indirect
```

Note:

```text
Prefer `-gpu software` or `-gpu swiftshader`. `swiftshader_indirect` is a legacy fallback.
```

## If User Says Build APK/AAB

Debug:

```bash
php artisan native:run android --build=debug
```

Release:

```bash
php artisan native:run android --build=release
```

Bundle:

```bash
php artisan native:run android --build=bundle
```

Before release:

```bash
npm run build
php artisan optimize:clear
```

Release checklist:

- `APP_ENV=production`.
- `APP_DEBUG=false`.
- Production API URL configured.
- No `10.0.2.2`.
- No local database credentials.
- Frontend assets built.
- Real device tested.

## If User Says NativePHP Command Missing

Likely causes:

- Package not installed.
- Wrong NativePHP package.
- Composer autoload stale.
- Running command outside Laravel project.
- Version changed command names.

Commands:

```bash
composer show nativephp/mobile
php artisan list
php artisan list | findstr native
composer dump-autoload
php artisan optimize:clear
```

Install:

```bash
composer require nativephp/mobile
php artisan native:install
```

## Automatic Safety Rules

GPT should always remember:

- NativePHP is still a Laravel app.
- Debug Laravel in browser before debugging native shell.
- Android emulator cannot use host `localhost`.
- Use `10.0.2.2` for emulator to host machine.
- Use `adb reverse` for physical USB device if useful.
- Clear config cache after `.env` changes.
- Check Laravel logs and `adb logcat`.
- Do not ship development URLs in production builds.
- Do not ship database passwords in mobile builds.
- Use API backend for real production mobile apps.

## Short Response Examples

User: NativePHP app blank screen.

GPT:

```text
Most likely it is either a Laravel exception, wrong APP_URL, or emulator networking. First run `php artisan serve --host=0.0.0.0 --port=8000` and open the app in a desktop browser. Then from Android use `http://10.0.2.2:8000`, clear cache with `php artisan optimize:clear`, and check `storage/logs/laravel.log` plus `adb logcat`.
```

User: POST not working in NativePHP.

GPT:

```text
First confirm whether this is a Blade form POST or JSON API POST. For Blade forms, make sure `@csrf` exists and the route appears in `php artisan route:list`. For API JSON, send `Accept: application/json` and `Content-Type: application/json`. If this runs in Android emulator, make sure generated URLs do not point to `localhost`; use `10.0.2.2` for host access.
```

User: NativePHP cannot connect to MySQL.

GPT:

```text
If MySQL is on your computer and the app runs in Android emulator, use `DB_HOST=10.0.2.2`, then run `php artisan config:clear`. For production mobile apps, do not connect directly to MySQL from the app; expose a Laravel API instead.
```


---

# Source: nativephp-advanced-knowledge.md

# NativePHP Advanced Knowledge

Use this file as deeper NativePHP training/reference data for Laravel mobile/native app planning, implementation, debugging, and release workflows.

## NativePHP Mental Model

NativePHP apps are Laravel apps running inside a native container.

Think in layers:

```text
Laravel application
Blade/Inertia/Livewire/API frontend
NativePHP bridge/runtime
Android/iOS native shell
Device OS features
Network/backend services
```

Debug order:

1. Laravel works in desktop browser.
2. Laravel works from emulator/device URL.
3. NativePHP command/build works.
4. Native shell/webview loads correct URL.
5. Device/native feature works.

Do not start by changing native files when the Laravel route itself fails.

## NativePHP Project Setup Flow

Typical flow:

```bash
composer create-project laravel/laravel app-name
cd app-name
composer require nativephp/mobile
php artisan native:install
npm install
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
php artisan native:run android --watch
```

If command names differ:

```bash
php artisan list | findstr native
composer show nativephp/mobile
```

NativePHP command behavior can vary by package version. Prefer the installed project's command list over memory.

## App URL Strategy

Development browser:

```env
APP_URL=http://127.0.0.1:8000
```

Android emulator reaching host Laravel server:

```env
APP_URL=http://10.0.2.2:8000
```

Physical device on same LAN:

```env
APP_URL=http://192.168.1.50:8000
```

Production:

```env
APP_URL=https://api.example.com
```

Important:

- `localhost` inside Android emulator means Android itself.
- `10.0.2.2` points from Android emulator to host computer.
- Do not ship `10.0.2.2` in production.
- Clear Laravel config after `.env` changes.

```bash
php artisan config:clear
php artisan optimize:clear
```

## NativePHP App Architecture Choices

### Server-Backed App

The mobile app talks to a remote Laravel backend API.

Good for:

- Production apps.
- Multi-device sync.
- Login/accounts.
- Payments.
- Admin-managed data.
- Push notifications.

Rules:

- Use token/session auth intentionally.
- Store only needed local data.
- Handle offline and retry states.
- Do not embed database credentials.

### Local Laravel App

The Laravel app runs locally inside the native app.

Good for:

- Local-first utilities.
- Internal tools.
- Offline workflows.
- Apps with local SQLite storage.

Rules:

- Plan backup/export/sync.
- Keep local database migrations stable.
- Avoid assuming a server queue worker always runs.

### Hybrid App

Local UI plus remote API sync.

Good for:

- Offline-capable business apps.
- Field data collection.
- Inventory/inspection apps.

Rules:

- Use sync state columns.
- Make sync idempotent.
- Handle conflicts.
- Store external IDs.

Example sync fields:

```text
local_id
remote_id
sync_status
last_synced_at
deleted_at
updated_at
```

## Routing In NativePHP

Blade/web routes still work:

```php
Route::get('/', DashboardController::class)->name('dashboard');
Route::resource('posts', PostController::class);
```

API routes still work:

```php
Route::apiResource('posts', PostApiController::class);
```

For mobile app navigation:

- Prefer named routes.
- Avoid hardcoded absolute local URLs.
- Use route helpers where possible.
- For external backend APIs, keep base URL in config.

Bad:

```php
return redirect('http://localhost:8000/posts');
```

Good:

```php
return redirect()->route('posts.index');
```

## Forms In NativePHP

Blade forms:

```blade
<form method="POST" action="{{ route('posts.store') }}">
    @csrf
    <input name="title" value="{{ old('title') }}">
    <button type="submit">Save</button>
</form>
```

Update:

```blade
<form method="POST" action="{{ route('posts.update', $post) }}">
    @csrf
    @method('PUT')
    <button type="submit">Update</button>
</form>
```

Delete:

```blade
<form method="POST" action="{{ route('posts.destroy', $post) }}">
    @csrf
    @method('DELETE')
    <button type="submit">Delete</button>
</form>
```

File upload:

```blade
<form method="POST" action="{{ route('files.store') }}" enctype="multipart/form-data">
    @csrf
    <input type="file" name="document">
    <button type="submit">Upload</button>
</form>
```

Common form problems:

- Missing `@csrf`.
- Wrong route name.
- Wrong HTTP method.
- Missing `enctype` for files.
- Validation errors not displayed.
- Session/cookie issue.
- Generated URL points to `localhost`.

## API Calls In NativePHP

If using JavaScript fetch:

```js
const response = await fetch('/api/posts', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ title: 'Native post' }),
});

const payload = await response.json();
```

With token:

```js
const response = await fetch(`${apiBaseUrl}/api/user`, {
  headers: {
    'Accept': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
});
```

Rules:

- Check HTTP status.
- Read response body on errors.
- Handle offline/network errors.
- Do not hardcode production secrets.
- Use HTTPS for production APIs.

Error handling:

```js
try {
  const response = await fetch(url, options);

  if (!response.ok) {
    const error = await response.text();
    throw new Error(error);
  }
} catch (error) {
  console.error(error);
}
```

## Authentication Strategy

### Session Auth

Good for:

- Blade-first apps.
- Webview app that behaves like a web app.

Watch:

- CSRF tokens.
- Cookie persistence.
- Cookie domain.
- Secure cookies.
- SameSite behavior.

Local development:

```env
SESSION_DRIVER=file
SESSION_SECURE_COOKIE=false
```

### Token Auth

Good for:

- API-first apps.
- Remote backend.
- Mobile login.

Rules:

- Store tokens using platform-secure storage if available.
- Provide logout/revoke.
- Refresh tokens safely if used.
- Do not store tokens in plain text files when avoidable.

### Production Warning

Never treat the mobile app as trusted. Anything inside an APK/IPA can be inspected.

## Database Strategy

### Remote Backend API

Recommended for most production apps:

```text
Native app -> HTTPS API -> Laravel backend -> database
```

Pros:

- Secure credentials.
- Centralized data.
- Easier updates.
- Better audit/logging.

### Direct MySQL In Development

Emulator:

```env
DB_HOST=10.0.2.2
DB_PORT=3306
```

Physical device:

```env
DB_HOST=<computer-lan-ip>
DB_PORT=3306
```

Warning:

```text
Do not ship direct production database credentials in a mobile app.
```

### SQLite Local Storage

Good for:

- Offline data.
- Local-only apps.
- Temporary cache.

Migration reminder:

```bash
php artisan migrate
```

Design:

- Keep local schema simple.
- Plan migrations between app versions.
- Plan sync if data must reach server.

## Assets And Vite

Development:

```bash
npm run dev
```

Production/release:

```bash
npm run build
```

Symptoms:

- Blank screen.
- Unstyled page.
- JavaScript not running.
- Vite manifest missing.

Fix:

```bash
npm install
npm run build
php artisan view:clear
```

Native/mobile note:

- Hot reload may require the device to reach the Vite server.
- Release builds should use built assets, not local Vite dev server.

## Storage And Files

Laravel storage:

```php
$path = $request->file('document')->store('documents');
```

Public files:

```bash
php artisan storage:link
```

Native considerations:

- Device file permissions may be needed for camera/gallery/files.
- Private documents should not be stored in public paths.
- Large files should upload in background or with progress when possible.
- Validate file size and MIME type.

Validation:

```php
$request->validate([
    'document' => ['required', 'file', 'max:10240'],
]);
```

## Device Permissions

Common mobile features need permissions:

- Camera.
- Photos/gallery.
- Location.
- Notifications.
- Files/documents.
- Microphone.

Rules:

- Ask permission only when needed.
- Explain why permission is needed in UI.
- Handle denied permission.
- Rebuild app after native permission/config changes.
- Use NativePHP-supported plugin/config mechanisms for the installed version.

## Native Plugins

Plugin rules:

- Check if plugin supports Android, iOS, or both.
- Register plugin if required.
- Rebuild after installing/registering plugin.
- Keep native permissions aligned with plugin needs.
- Test on real device, not only emulator.

Command noted in NativePHP docs:

```bash
php artisan native:plugin:register
```

## Background Work

Laravel queues:

```env
QUEUE_CONNECTION=sync
```

For development/simple local app, `sync` is easiest.

For backend server:

```env
QUEUE_CONNECTION=database
```

Run worker:

```bash
php artisan queue:work
```

Mobile warning:

- Do not assume a mobile app can run a long-lived Laravel queue worker reliably.
- Push reliable background work to backend server when possible.
- Use native background capabilities only when supported and tested.

## Notifications

Types:

- In-app notifications.
- Email notifications.
- Push notifications.
- Local device notifications.

Rules:

- For push, use a backend service and device token registration.
- Store device tokens per user/device.
- Handle token refresh.
- Do not assume push delivery is guaranteed.

Device token table idea:

```text
id
user_id
platform
token
last_seen_at
created_at
updated_at
```

## Offline And Sync

Offline app rules:

- Detect offline state.
- Queue local changes.
- Retry sync later.
- Show sync status.
- Resolve conflicts.
- Use idempotent API endpoints.

Local record fields:

```text
sync_status: pending, synced, failed
remote_id
last_synced_at
sync_error
```

Conflict strategies:

- Last write wins.
- Server wins.
- Manual review.
- Field-level merge.

Choose intentionally.

## Build And Release

Debug build:

```bash
php artisan native:run android --build=debug
```

Release build:

```bash
php artisan native:run android --build=release
```

Bundle:

```bash
php artisan native:run android --build=bundle
```

Before release:

```bash
npm run build
php artisan optimize:clear
```

Release checklist:

- `APP_ENV=production`.
- `APP_DEBUG=false`.
- Production URLs configured.
- No `localhost`.
- No `10.0.2.2`.
- No local database credentials.
- Assets built.
- Permissions configured.
- App icon/splash/name correct.
- Real device tested.
- Login/logout tested.
- Offline/network failure tested.
- Form POST/API POST tested.
- File upload tested if used.

## Android Signing Concepts

Release Android apps usually require signing.

Concepts:

- Keystore.
- Key alias.
- Keystore password.
- Key password.
- APK/AAB signing.

Rules:

- Back up keystore securely.
- Do not commit keystore passwords.
- Losing production signing key can block updates depending on store setup.
- Use environment variables or secure CI secrets for signing data.

## iOS Release Concepts

iOS usually involves:

- Apple Developer account.
- Bundle identifier.
- Provisioning profile.
- Signing certificate.
- App Store Connect.
- Real device testing.

Rules:

- Test on real iOS device.
- Configure permissions descriptions.
- Keep bundle ID stable.

## Debugging NativePHP

Laravel logs:

```text
storage/logs/laravel.log
```

Android logs:

```bash
adb logcat
adb logcat | findstr AndroidRuntime
adb logcat | findstr NativePHP
```

Laravel commands:

```bash
php artisan about
php artisan route:list
php artisan optimize:clear
php artisan config:clear
```

NativePHP commands:

```bash
php artisan list | findstr native
php artisan native:run android --watch
```

## Common Errors

### App opens blank

Likely:

- Laravel exception.
- Wrong URL.
- Missing assets.
- JavaScript error.
- Native shell issue.

Fix:

```bash
php artisan optimize:clear
npm run build
adb logcat
```

### POST returns 419

Likely:

- Missing CSRF token.
- Session issue.
- Cookie issue.

Fix:

```blade
@csrf
```

Check session config:

```env
SESSION_DRIVER=file
SESSION_SECURE_COOKIE=false
```

### API returns 401

Likely:

- Missing token.
- Expired token.
- Wrong guard.
- User logged out.

Check:

```text
Authorization header
auth middleware
token storage
backend logs
```

### API returns 422

Likely:

- Validation failed.

Fix:

```text
Read response JSON.
Display validation errors.
Check required fields.
```

### Cannot connect to server

Emulator:

```text
Use 10.0.2.2.
```

Physical device:

```text
Use computer LAN IP or adb reverse.
```

Command:

```bash
adb reverse tcp:8000 tcp:8000
```

## NativePHP GPT Answer Rules

When answering NativePHP questions:

- Mention that NativePHP is still Laravel first if relevant.
- Ask whether it is Android emulator, physical Android, iOS simulator, or real iOS device when networking matters.
- For app loading issues, start with browser verification.
- For POST issues, distinguish Blade form from JSON API.
- For database issues, warn against shipping DB credentials.
- For production builds, warn against `10.0.2.2` and `APP_DEBUG=true`.
- For command issues, suggest `php artisan list | findstr native`.
- For version-specific behavior, tell user to check installed NativePHP command list.

