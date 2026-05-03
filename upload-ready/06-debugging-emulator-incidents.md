# debugging emulator incidents


---

# Source: debugging-notes.md

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


---

# Source: emulator-fixes.md

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


---

# Source: realworld-problem-solving.md

# Real-World Problem Solving

Use this file to make GPT better at diagnosing practical software problems, not just remembering commands.

## First Response Pattern

When a user reports a problem, collect facts before guessing:

- What changed recently?
- What exact command/action caused the issue?
- What exact error message appeared?
- Does it happen locally, staging, production, emulator, or real device?
- Is it reproducible every time?
- Which logs show the error?
- Is the problem user-specific, data-specific, environment-specific, or global?

Good debugging response:

```text
This looks like a config/cache or connection-layer issue. First confirm the exact URL, clear Laravel config cache, then check the Laravel log and network path from the device.
```

Avoid:

```text
Try reinstalling everything.
```

## Layered Diagnosis

Find the layer:

```text
User interface -> JavaScript -> HTTP request -> route -> middleware -> controller -> validation -> service -> database/API -> queue -> notification/storage
```

For mobile/NativePHP:

```text
Native shell -> webview -> Laravel app -> network -> API/database -> device permissions -> OS logs
```

For deployment:

```text
DNS -> SSL -> web server -> PHP-FPM -> Laravel -> cache/session/queue -> database -> filesystem
```

## Symptoms To Causes

Blank page:

- PHP exception hidden by `APP_DEBUG=false`.
- JavaScript error.
- Missing built assets.
- Wrong base URL.
- Native webview cannot reach server.
- Route returns empty response.

Slow page:

- N+1 queries.
- Missing index.
- External API call inside request.
- Large unpaginated result.
- Heavy Blade loop.
- Queue work done synchronously.
- Cache missing or ineffective.

Login not working:

- Wrong credentials.
- Session not persisting.
- CSRF issue.
- Cookie domain/secure/SameSite issue.
- User inactive or policy/middleware denial.
- Config cache still has old `.env`.

File upload fails:

- Form missing `enctype="multipart/form-data"`.
- Validation max size too low.
- PHP `upload_max_filesize` or `post_max_size`.
- Storage folder permission.
- Missing `php artisan storage:link`.
- Disk config wrong.

Email not sending:

- Wrong mail driver.
- SMTP credentials wrong.
- Firewall blocks SMTP port.
- Mail queued but worker not running.
- Message sent to log/mailpit locally.
- Provider sandbox mode.

Queue not working:

- Worker not running.
- `QUEUE_CONNECTION` wrong.
- Jobs table missing.
- Failed jobs not retried.
- Worker using old code after deploy.
- Job timeout too low.

## Evidence Ranking

Trust in this order:

1. Exact logs and stack traces.
2. Reproducible minimal steps.
3. Database state.
4. Network requests/responses.
5. Configuration files and environment values.
6. Recent commits/deployments.
7. Memory/guess.

## Minimal Reproduction

Reduce the problem:

- One route.
- One controller method.
- One database record.
- One request payload.
- One environment.
- One failing command.

Example:

```php
Route::get('/debug-db', function () {
    return DB::table('users')->count();
});
```

Remove this route after debugging.

## Fix Strategy

Prefer small, testable fixes:

- Fix root cause, not only symptom.
- Add validation for bad input.
- Add authorization for protected actions.
- Add index for proven slow query.
- Move slow work to queue.
- Add retry/backoff for unreliable external APIs.
- Add logging around uncertain integration points.
- Add a regression test for a bug that can return.

## When To Roll Back

Rollback is reasonable when:

- Production is broken.
- Impact is high.
- Root cause is unclear.
- A recent deploy likely caused it.
- Rollback is safer than live debugging.

After rollback:

- Preserve logs.
- Identify bad commit/config.
- Reproduce in staging.
- Write test.
- Redeploy fixed version.

## Real-World Laravel Triage

For a Laravel production issue:

```bash
php artisan about
php artisan route:list
php artisan queue:failed
php artisan schedule:list
```

Check:

- `storage/logs/laravel.log`
- web server logs
- queue worker logs
- database slow query log
- recent deploy changes
- `.env` differences
- disk space
- failed jobs

## Decision Rules

- If data can be lost, back up before changing.
- If a command deletes or rewrites data, stop and confirm.
- If a fix touches authentication, payment, payroll, or permissions, add tests.
- If a bug affects many users, prefer rollback or feature flag disable.
- If a query is slow, prove with `EXPLAIN` before adding indexes.
- If emulator cannot connect, verify the URL from the emulator perspective.
- If `.env` changed, clear config cache.

## GPT Behavior Rules

For real-world problems, GPT should:

- Ask for exact error if missing.
- Mention the most likely layer.
- Give a short ordered checklist.
- Include concrete commands.
- Warn before destructive steps.
- Avoid vague advice like "check everything".
- Avoid assuming `localhost` works inside Android emulator.
- Prefer Laravel-native tools and conventions.
- Suggest tests for repeated or critical bugs.


---

# Source: incident-runbooks.md

# Incident Runbooks

Use this file to help GPT respond quickly to production and real-user problems.

## Production Site Down

Immediate steps:

1. Confirm from outside the server.
2. Check recent deploys.
3. Check web server/PHP logs.
4. Check Laravel logs.
5. Check database connectivity.
6. Check disk space.
7. Roll back if recent deploy caused outage.

Commands:

```bash
curl -I https://example.com
tail -f storage/logs/laravel.log
df -h
free -m
php artisan about
```

Common causes:

- Syntax/runtime error after deploy.
- `.env` missing or wrong.
- Config cache stale.
- Database down.
- Disk full.
- File permissions wrong.
- PHP extension missing.

## Database Slow Or Down

Immediate checks:

- Is database reachable?
- Is CPU high?
- Are there long-running queries?
- Did a migration or report start?
- Did traffic spike?

MySQL:

```sql
SHOW FULL PROCESSLIST;
SHOW ENGINE INNODB STATUS;
```

Laravel:

```bash
php artisan queue:failed
```

Fix options:

- Kill clearly bad long-running query.
- Disable expensive report route/job.
- Add missing index after testing.
- Move heavy report to queue.
- Restore from backup only when data is damaged.

## Queue Backlog

Symptoms:

- Emails delayed.
- Imports stuck.
- Notifications not sent.
- `jobs` table growing.

Checks:

```bash
php artisan queue:failed
php artisan queue:work --tries=3
```

Fix:

- Start/restart workers.
- Check failed job exception.
- Increase worker count if safe.
- Fix bad job and retry.
- Do not blindly retry all jobs if the bug still exists.

Retry:

```bash
php artisan queue:retry all
```

Flush failed jobs only after deciding they are not needed:

```bash
php artisan queue:flush
```

## Payment Failure

Rules:

- Never charge twice without verifying provider state.
- Use idempotency keys where provider supports them.
- Log provider request ID, transaction ID, order ID.
- Keep payment state machine explicit.
- Reconcile webhook and direct response.

Statuses:

```text
pending
authorized
paid
failed
cancelled
refunded
partially_refunded
```

Debug:

- Check application logs.
- Check payment provider dashboard.
- Check webhook delivery logs.
- Check duplicate requests.
- Check timeout behavior.

## Email Delivery Incident

Checks:

- Is mail queued?
- Is worker running?
- Are SMTP/API credentials valid?
- Is provider rejecting messages?
- Are messages going to spam?
- Is DNS configured: SPF, DKIM, DMARC?

Temporary mitigation:

- Switch to backup provider if configured.
- Retry failed jobs after fixing cause.
- Notify users through in-app notification if email is delayed.

## File Storage Incident

Symptoms:

- Upload fails.
- Images broken.
- Downloads return 404/403.
- Disk full.

Checks:

```bash
php artisan storage:link
df -h
```

Laravel checks:

- Disk configured in `config/filesystems.php`.
- `storage` folder writable.
- Public symlink exists.
- Cloud credentials valid.
- File visibility public/private matches route.

## Security Incident

If secrets are leaked:

1. Revoke/rotate leaked secrets.
2. Remove from Git history only if required, but rotation matters most.
3. Check logs for misuse.
4. Deploy new secrets.
5. Document impact.

If account compromise suspected:

- Force password reset.
- Revoke tokens/sessions.
- Review recent actions.
- Add rate limiting or MFA if missing.

## Bad Deploy

Mitigation:

```bash
php artisan down
git switch previous-release
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan optimize:clear
php artisan up
```

Better production systems use release folders and symlink switching.

After incident:

- Write root cause.
- Add regression test.
- Add monitor/alert if possible.
- Improve deploy checklist.

