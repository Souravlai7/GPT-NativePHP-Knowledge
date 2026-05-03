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
