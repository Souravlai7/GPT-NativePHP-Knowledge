# security integrations deployment


---

# Source: api-security-knowledge.md

# API And Security Knowledge

Use this file as API design, authentication, validation, and web security training/reference data.

## REST API Basics

Common methods:

```text
GET     read
POST    create
PUT     replace/update
PATCH   partial update
DELETE  delete
```

Common status codes:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthenticated
403 Forbidden
404 Not Found
409 Conflict
422 Validation Error
429 Too Many Requests
500 Server Error
```

Resource examples:

```text
GET /api/posts
POST /api/posts
GET /api/posts/{id}
PATCH /api/posts/{id}
DELETE /api/posts/{id}
```

## Laravel API Response Pattern

```php
return response()->json([
    'data' => $post,
]);
```

Created:

```php
return response()->json([
    'data' => $post,
], 201);
```

No content:

```php
return response()->noContent();
```

Validation:

```php
$validated = $request->validate([
    'email' => ['required', 'email'],
    'password' => ['required', 'string', 'min:8'],
]);
```

## Authentication

Session auth:

```text
Best for traditional web apps.
Uses cookies and CSRF protection.
```

Token auth:

```text
Best for APIs, mobile apps, service integrations.
Use short-lived tokens where possible.
Revoke tokens when compromised.
```

Laravel helpers:

```php
$request->user();
auth()->user();
auth()->id();
```

## Authorization

Policy:

```php
public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}
```

Controller:

```php
$this->authorize('update', $post);
```

Blade:

```blade
@can('update', $post)
    Edit
@endcan
```

## Security Rules

- Never trust user input.
- Validate all request data.
- Escape output unless intentionally rendering trusted HTML.
- Use parameter binding, not string-concatenated SQL.
- Keep `APP_DEBUG=false` in production.
- Do not commit `.env`.
- Do not store passwords as plain text.
- Use HTTPS in production.
- Use rate limiting for login, OTP, and public APIs.
- Avoid exposing stack traces to users.
- Log enough context to debug, but never secrets.

## SQL Injection

Bad:

```php
DB::select("SELECT * FROM users WHERE email = '$email'");
```

Good:

```php
DB::select('SELECT * FROM users WHERE email = ?', [$email]);
User::where('email', $email)->first();
```

## XSS

Safe Blade output:

```blade
{{ $userInput }}
```

Unsafe if content is not trusted:

```blade
{!! $userInput !!}
```

## CSRF

Forms need:

```blade
@csrf
```

APIs commonly use token authentication instead of browser CSRF tokens.

## Rate Limiting

Route:

```php
Route::middleware('throttle:60,1')->group(function () {
    Route::get('/api/posts', [PostController::class, 'index']);
});
```

Use stricter limits for:

```text
login
password reset
OTP send/verify
payment attempts
public search endpoints
```

## File Upload Security

Rules:

- Validate file type.
- Validate file size.
- Store uploaded files outside executable paths when possible.
- Generate safe filenames.
- Do not trust original filenames.
- Scan high-risk uploads if needed.

Laravel:

```php
$validated = $request->validate([
    'avatar' => ['required', 'image', 'max:2048'],
]);

$path = $request->file('avatar')->store('avatars', 'public');
```

## Secrets

Do not commit:

```text
.env
API keys
database passwords
private keys
OAuth client secrets
JWT secrets
```

Rotate secrets if leaked.

## CORS

CORS controls which browser origins can call an API.

Development may allow local origins:

```text
http://localhost:3000
http://127.0.0.1:5173
```

Production should allow only known frontend domains.


---

# Source: integration-patterns.md

# Integration Patterns

Use this file for real-world integrations: payments, webhooks, email, SMS, files, imports, exports, and external APIs.

## External API Rules

- Set timeouts.
- Handle non-200 responses.
- Retry transient failures.
- Do not retry permanent failures blindly.
- Log request IDs, not secrets.
- Validate incoming and outgoing payloads.
- Use queues for slow or unreliable APIs.

Laravel HTTP client:

```php
$response = Http::timeout(10)
    ->retry(3, 200)
    ->acceptJson()
    ->post($url, $payload);

if ($response->failed()) {
    report(new RuntimeException($response->body()));
}
```

## Webhooks

Webhook rules:

- Verify signature.
- Store event ID.
- Make handling idempotent.
- Return 2xx only after accepting the event.
- Process slow work in queue.
- Keep raw payload for debugging if safe.

Example table fields:

```text
id
provider
event_id
event_type
payload
processed_at
created_at
```

Idempotency:

```php
if (WebhookEvent::where('event_id', $eventId)->exists()) {
    return response()->noContent();
}
```

## Payments

Payment rules:

- Use idempotency keys.
- Do not trust frontend payment status.
- Confirm status with provider or webhook.
- Store provider transaction ID.
- Keep clear payment state.
- Never log card data.
- Handle refunds and partial refunds.

Order/payment states:

```text
order: draft, pending_payment, paid, cancelled, fulfilled
payment: pending, authorized, captured, failed, refunded
```

## Imports

Import rules:

- Validate file type and size.
- Store uploaded file.
- Parse in queued job for large files.
- Validate each row.
- Report row-level errors.
- Use transactions carefully.
- Avoid loading huge files fully into memory.

Import flow:

```text
Upload -> create import record -> queue parser job -> validate rows -> write data -> mark completed/failed -> notify user
```

## Exports

Export rules:

- Queue large exports.
- Store generated file.
- Notify user when ready.
- Expire old export files.
- Use streaming for medium exports.

Flow:

```text
Request export -> create export record -> queue job -> generate CSV/XLSX/PDF -> store -> notify user
```

## Email

Rules:

- Queue email in production.
- Use templates.
- Avoid sending duplicate transactional emails.
- Keep unsubscribe links for marketing emails.
- Configure SPF, DKIM, DMARC.

Laravel:

```php
Mail::to($user)->queue(new WelcomeMail($user));
```

## SMS/OTP

Rules:

- Rate limit send attempts.
- Expire OTP quickly.
- Store hashed OTP if possible.
- Limit verification attempts.
- Do not reveal whether phone/email exists.
- Log provider message ID.

OTP fields:

```text
user_id
destination
code_hash
expires_at
attempts
verified_at
```

## File Storage

Rules:

- Validate upload.
- Generate safe filename.
- Store metadata in database.
- Use private storage for sensitive files.
- Use signed URLs for private downloads.
- Virus scan high-risk files if required.

Signed download:

```php
URL::temporarySignedRoute('files.download', now()->addMinutes(10), [
    'file' => $file->id,
]);
```

## Native/Mobile API Integration

Mobile rules:

- Expect network failures.
- Add retry only for safe/idempotent operations.
- Show offline/error states.
- Do not store API secrets in the app.
- Use token revocation/logout.
- Avoid direct production database access from mobile apps.


---

# Source: deployment-ops-knowledge.md

# Deployment And Operations Knowledge

Use this file as deployment, server, Linux, Windows, environment, queue worker, and operations training/reference data.

## Laravel Deployment Checklist

```bash
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan queue:restart
```

Production `.env`:

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://example.com
LOG_LEVEL=warning
```

Writable folders:

```text
storage/
bootstrap/cache/
```

## Zero-Downtime Ideas

- Put app in maintenance mode only if necessary.
- Build assets before switching release.
- Run migrations carefully.
- Avoid destructive migrations during high traffic.
- Restart queue workers after code deploy.
- Keep backups before risky database changes.

## Queue Workers

Run locally:

```bash
php artisan queue:work
```

Restart after deploy:

```bash
php artisan queue:restart
```

Supervisor concept:

```text
Supervisor/systemd keeps queue workers running in production.
Workers should restart automatically after failure.
```

Worker flags:

```bash
php artisan queue:work --tries=3
php artisan queue:work --queue=high,default
php artisan queue:work --timeout=120
```

## Cron Scheduler

Laravel scheduler:

```bash
php artisan schedule:run
```

Crontab:

```cron
* * * * * cd /path/to/app && php artisan schedule:run >> /dev/null 2>&1
```

Scheduled command:

```php
Schedule::command('reports:send')->daily();
Schedule::job(new CleanupTempFiles)->hourly();
```

## Linux Commands

Files:

```bash
ls -la
pwd
cp source dest
mv old new
rm file
mkdir folder
```

Logs:

```bash
tail -f storage/logs/laravel.log
journalctl -u service-name -f
```

Permissions:

```bash
chmod -R ug+rw storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

Processes:

```bash
ps aux
top
htop
kill PID
```

Network:

```bash
curl -I https://example.com
ping example.com
netstat -tulpn
ss -tulpn
```

## Windows Commands

PowerShell:

```powershell
Get-ChildItem
Get-Content .\storage\logs\laravel.log -Tail 100 -Wait
Get-Process
Get-NetTCPConnection -LocalPort 8000
ipconfig
```

Find process using port:

```powershell
Get-NetTCPConnection -LocalPort 8000 | Select-Object OwningProcess
Get-Process -Id <pid>
```

## Environment Variables

Rules:

- `.env` is environment-specific.
- Do not commit `.env`.
- Run `php artisan config:clear` after changing `.env` locally.
- Run `php artisan config:cache` in production.

Common production variables:

```env
APP_ENV=production
APP_DEBUG=false
CACHE_STORE=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
LOG_CHANNEL=stack
```

## Backups

Backup before:

- Large migrations.
- Deleting records.
- Production data imports.
- Changing payment, payroll, or accounting data.

MySQL dump:

```bash
mysqldump -u user -p database_name > backup.sql
```

Restore:

```bash
mysql -u user -p database_name < backup.sql
```

## Monitoring

Watch:

- Error rate.
- Slow requests.
- Queue length.
- Failed jobs.
- Database CPU/load.
- Disk space.
- Memory usage.
- SSL certificate expiry.

Laravel places to inspect:

```text
storage/logs/laravel.log
failed_jobs table
queue worker logs
web server logs
database slow query log
```

