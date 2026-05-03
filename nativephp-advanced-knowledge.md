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
