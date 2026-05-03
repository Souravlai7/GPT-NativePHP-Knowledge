# Debugging Notes

Use this file as Laravel, PHP, Composer, MySQL, NativePHP, Android, and Windows troubleshooting training/reference data.

## General Debugging Flow

Use this order:

1. Reproduce the issue.
2. Read the exact error message.
3. Check the newest log entry.
4. Identify the layer: PHP, Laravel, database, web server, frontend, Android, NativePHP, OS, network.
5. Clear caches only when cache/config could be stale.
6. Make one change at a time.
7. Re-test.

Important logs:

```text
storage/logs/laravel.log
browser devtools console
terminal running php artisan serve
terminal running npm run dev
adb logcat
MySQL error log
web server error log
```

## Laravel Debug Commands

```bash
php artisan about
php artisan route:list
php artisan config:show app
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear
composer dump-autoload
php artisan test
```

Check app in Tinker:

```bash
php artisan tinker
config('app.url')
DB::connection()->getPdo()
User::count()
```

## Laravel Log Debugging

Log file:

```text
storage/logs/laravel.log
```

Add logs:

```php
logger()->debug('Reached checkout', ['user_id' => auth()->id()]);
Log::info('Payment response', ['response' => $response]);
Log::error('Import failed', ['exception' => $e]);
```

Do not log:

```text
passwords
tokens
private keys
credit card data
session cookies
```

## Composer SSL Error curl error 60

Common cause:

```text
Antivirus HTTPS scanning, missing CA certificate, outdated PHP CA bundle, or corporate proxy.
```

Fix options:

```text
Disable antivirus HTTPS/SSL scan.
Install/update CA certificate bundle.
Configure PHP openssl.cafile.
Configure Composer cafile.
Check proxy settings.
```

Commands:

```bash
composer diagnose
php -i
```

PHP ini setting example:

```ini
openssl.cafile="C:\path\to\cacert.pem"
curl.cainfo="C:\path\to\cacert.pem"
```

Avoid insecure workaround except temporary local diagnosis:

```bash
composer config -g secure-http false
```

Do not leave `secure-http false` enabled.

## Class Not Found

Common causes:

```text
Wrong namespace.
Wrong file path or class name casing.
Composer autoload not updated.
Missing import/use statement.
Package not installed.
```

Fix:

```bash
composer dump-autoload
php artisan optimize:clear
```

Check:

```php
use App\Models\User;
use App\Http\Controllers\PostController;
```

## Route Not Found

Debug:

```bash
php artisan route:list
php artisan route:list --name=profile
php artisan route:clear
```

Common causes:

```text
Route name typo.
Route file not loaded.
Route cached in production.
HTTP method mismatch.
Missing route parameter.
```

Example:

```php
Route::get('/posts/{post}', [PostController::class, 'show'])->name('posts.show');
route('posts.show', $post);
```

## 419 Page Expired

Common causes:

```text
Missing CSRF token.
Session not writable.
Session domain/cookie mismatch.
APP_KEY changed.
Stale cached config.
```

Fix:

```blade
@csrf
```

Commands:

```bash
php artisan optimize:clear
```

Check `.env`:

```env
SESSION_DRIVER=file
SESSION_DOMAIN=
SESSION_SECURE_COOKIE=false
```

## 403 Forbidden

Common causes:

```text
Policy denies action.
Gate denies action.
Middleware denies request.
Web server file permission issue.
```

Debug:

```php
Gate::allows('update', $post);
$this->authorize('update', $post);
```

## 404 Not Found

Common causes:

```text
Route missing.
Wrong HTTP method.
Route model binding cannot find record.
Web server rewrite config missing.
Wrong APP_URL or base path.
```

Debug:

```bash
php artisan route:list
```

## 500 Server Error

Fix flow:

```text
Set APP_DEBUG=true locally.
Check storage/logs/laravel.log.
Check web server/PHP error log.
Clear config cache after .env changes.
```

Commands:

```bash
php artisan optimize:clear
composer dump-autoload
```

## Migration Problems

Check status:

```bash
php artisan migrate:status
```

Rollback:

```bash
php artisan migrate:rollback
php artisan migrate:rollback --step=1
```

Fresh local database:

```bash
php artisan migrate:fresh --seed
```

Common causes:

```text
Table already exists.
Column already exists.
Foreign key type mismatch.
Foreign key references missing table.
Migration order wrong.
Index name too long.
```

Foreign key types must match:

```php
$table->id(); // unsigned big integer
$table->foreignId('user_id')->constrained();
```

## MySQL Connection Errors

Common errors:

```text
SQLSTATE[HY000] [1045] Access denied
SQLSTATE[HY000] [2002] Connection refused
Unknown database
Base table or view not found
```

Check `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=root
DB_PASSWORD=
```

After editing `.env`:

```bash
php artisan config:clear
php artisan optimize:clear
```

Test:

```bash
php artisan tinker
DB::connection()->getPdo()
```

## NPM And Vite Problems

Install:

```bash
npm install
npm run dev
```

Build:

```bash
npm run build
```

Common fixes:

```bash
php artisan view:clear
npm install
npm run dev
```

Symptoms:

```text
CSS not loading.
JS not loading.
Vite manifest not found.
Hot reload not working.
Blank page due to JavaScript error.
```

For production:

```bash
npm run build
```

## Queue Problems

Worker not processing:

```bash
php artisan queue:work
php artisan queue:failed
```

Database queue setup:

```bash
php artisan queue:table
php artisan migrate
```

After deployment:

```bash
php artisan queue:restart
```

Common causes:

```text
QUEUE_CONNECTION wrong.
Worker not running.
Failed job not retried.
Old worker still running old code.
Job throws exception.
Missing database jobs table.
```

## File Permission Problems

Laravel must write:

```text
storage/
bootstrap/cache/
```

Linux fix:

```bash
chmod -R ug+rw storage bootstrap/cache
```

Windows checks:

```text
Folder not read-only.
User has write permission.
Antivirus not blocking generated files.
```

## Android Emulator Not Opening

Common causes:

```text
GPU issue.
Corrupted AVD snapshot.
Wrong SDK path.
Missing emulator package.
Hardware acceleration issue.
```

Fix:

```bash
emulator -list-avds
emulator -avd <name> -no-snapshot-load
emulator -avd <name> -gpu software
emulator -avd <name> -gpu swiftshader
```

Legacy fallback:

```bash
emulator -avd <name> -gpu swiftshader_indirect
```

## ADB Device Offline

Fix:

```bash
adb kill-server
adb start-server
adb devices
```

Physical device:

```text
Unlock phone.
Enable Developer Options.
Enable USB debugging.
Accept RSA prompt.
Try another USB cable/port.
Revoke USB debugging authorizations and reconnect.
```

## Laravel Not Accessible In Android Emulator

Host command:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

Emulator URL:

```text
http://10.0.2.2:8000
```

Do not use from Android emulator:

```text
http://127.0.0.1:8000
http://localhost:8000
```

Those point to the emulator itself, not the host computer.

## NativePHP Debugging

Run:

```bash
php artisan native:run android
php artisan native:run android --watch
```

Check:

```bash
adb devices
adb logcat
php artisan optimize:clear
```

Common NativePHP blank screen causes:

```text
Laravel app error.
Wrong APP_URL.
Device cannot reach host.
Vite assets missing.
JavaScript runtime error.
Native build/plugin error.
```

## Windows PowerShell Profile Warning

Error:

```text
profile.ps1 cannot be loaded because running scripts is disabled on this system.
```

Meaning:

```text
PowerShell tried to load the user profile script, but execution policy blocked it.
Commands can still run if the actual command succeeds.
```

Fix options:

```powershell
Get-ExecutionPolicy
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Only change execution policy if you understand the security impact.

## Debugging Checklist

- Read the exact error.
- Check Laravel log.
- Check terminal output.
- Check browser console.
- Check route list.
- Clear Laravel caches after `.env` or route/config changes.
- Confirm database credentials.
- Confirm migrations ran.
- Confirm queue worker is running.
- Confirm assets are built or Vite is running.
- For emulator, use `10.0.2.2`.
- For physical Android USB, try `adb reverse`.
- For NativePHP, prove the Laravel app works in browser first.
