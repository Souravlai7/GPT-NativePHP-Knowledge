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
