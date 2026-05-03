# GPT Conversation Starters

Use these as custom GPT conversation starters.

## Best 4 Starters

```text
Help me debug my NativePHP Android app not loading.
```

```text
Design an offline-first NativePHP app with SQLite local storage and Laravel API sync.
```

```text
Help me fix a Laravel API, database, or validation error from this log.
```

```text
Review my Laravel/NativePHP feature plan and suggest the safest architecture.
```

## NativePHP Starters

```text
My NativePHP app shows a blank screen. Walk me through the debug steps.
```

```text
My Laravel app works in browser but not in NativePHP Android emulator. What should I check?
```

```text
How do I run and debug a NativePHP app on Android emulator?
```

```text
How should I configure APP_URL for NativePHP in browser, Android emulator, physical device, and production?
```

```text
My NativePHP POST request is not working. Help me check Blade form, CSRF, route, and API issues.
```

```text
My NativePHP app cannot connect to MySQL/API from Android emulator. Help me fix the network setup.
```

```text
Create a NativePHP release checklist for Android APK/AAB.
```

## Offline And Local DB Starters

```text
Design a local SQLite database structure for offline NativePHP data.
```

```text
Help me create sync fields like local_uuid, remote_id, sync_status, and last_synced_at.
```

```text
Design create/update/delete sync flow for offline NativePHP records.
```

```text
How should I handle sync conflicts between local data and server data?
```

```text
Create a retry/backoff strategy for failed offline sync operations.
```

```text
How do I check internet connection properly before syncing data?
```

## API Starters

```text
Help me design a Laravel API for a NativePHP mobile app.
```

```text
Create a safe API client pattern with timeout, token auth, and error handling.
```

```text
My API returns 401/403/422/500. Help me debug it.
```

```text
How should I handle token storage and logout in a NativePHP mobile app?
```

```text
Design idempotent API endpoints for mobile sync.
```

## Payment Starters

```text
Design a safe payment flow for a NativePHP app with Laravel backend verification.
```

```text
How do I prevent duplicate payment charges with idempotency keys?
```

```text
Help me design payment webhook handling in Laravel.
```

```text
My payment status is unclear after internet loss. What should the app do?
```

```text
Create database fields for orders, payments, provider IDs, webhook events, and payment status.
```

## Socket And Realtime Starters

```text
Help me set up Laravel Reverb and Echo for realtime notifications.
```

```text
My socket connects but events are not received. Help me debug channel names, queues, and listeners.
```

```text
My Laravel Reverb socket fails in Android emulator. Help me fix host, port, and 10.0.2.2 settings.
```

```text
Explain public, private, and presence channels in Laravel broadcasting.
```

```text
Help me debug /broadcasting/auth for private channels.
```

## Laravel Starters

```text
Help me create a Laravel CRUD feature with validation, policy, routes, controller, migration, and tests.
```

```text
My Laravel POST request returns 419. Help me fix CSRF/session issues.
```

```text
My Laravel route is not found. Help me check routes, names, methods, and route cache.
```

```text
Help me move slow Laravel work into a queue job.
```

```text
Review this Laravel controller and suggest a cleaner architecture.
```

## MySQL And Performance Starters

```text
Analyze this slow Laravel query and suggest indexes using EXPLAIN.
```

```text
Help me fix N+1 queries in this Eloquent code.
```

```text
Design MySQL indexes for this Laravel table and query pattern.
```

```text
Help me choose between paginate, simplePaginate, and cursorPaginate.
```

```text
Optimize a Laravel dashboard with cached aggregates and queued reports.
```

## Deployment And Production Starters

```text
Create a Laravel production deployment checklist.
```

```text
My production Laravel app is down. Give me an incident checklist.
```

```text
My Laravel queue is not processing jobs in production. Help me debug it.
```

```text
Help me safely roll back a bad Laravel deployment.
```

```text
What should I check before releasing a NativePHP app to users?
```

## Debugging Starters

```text
I will paste an error log. Tell me the likely cause and exact fix steps.
```

```text
Help me separate whether this bug is Laravel, database, frontend, NativePHP, emulator, or network related.
```

```text
My file upload fails in Laravel/NativePHP. Help me check form, validation, storage, and permissions.
```

```text
Composer shows curl error 60. Help me fix SSL/certificate issues safely.
```

```text
My Android emulator shows device offline. Help me fix ADB.
```

## Security Starters

```text
Review this Laravel API flow for security issues.
```

```text
Help me protect private files in Laravel and NativePHP.
```

```text
What secrets should never be stored in a NativePHP mobile app?
```

```text
Help me add validation, authorization, and rate limiting to this endpoint.
```

## More Best Starters

```text
My NativePHP app works in browser but fails on Android emulator. Diagnose the likely layer.
```

```text
Create a safe payment flow for a NativePHP app with Laravel backend verification and webhooks.
```

```text
Help me set up Laravel Reverb sockets for a NativePHP app and debug emulator networking.
```

```text
Analyze this slow Laravel/MySQL query and suggest indexes using EXPLAIN.
```

## NativePHP Advanced Starters

```text
Explain the NativePHP mental model: Laravel app, native shell, webview, network, logs, and device features.
```

```text
Help me choose between server-backed, local-only, and hybrid NativePHP app architecture.
```

```text
Help me design NativePHP app permissions for camera, files, notifications, and location.
```

```text
My NativePHP command is missing. Help me check package install, artisan command list, and version differences.
```

```text
Explain why I should not ship direct production database credentials in a NativePHP mobile app.
```

## Extra Offline And Sync Starters

```text
Design a sync_operations table for offline changes and retries.
```

```text
How should I pull server changes using updated_after and server timestamps?
```

```text
Help me decide when to use SQLite, cache, filesystem storage, secure storage, localStorage, or IndexedDB.
```

```text
Design conflict handling using HTTP 409, version fields, server wins, client wins, and manual merge.
```

```text
Design a safe logout flow that clears local data and tokens when needed.
```

## Extra API Starters

```text
Create API response patterns using Laravel API resources and pagination metadata.
```

```text
Design a /health endpoint for internet checks and sync readiness.
```

```text
Create a reusable API wrapper for NativePHP with token auth, JSON parsing, timeout, and error handling.
```

```text
Help me handle API 401, 403, 404, 409, 422, 429, and 500 responses differently in a NativePHP app.
```

## Extra Payment Starters

```text
Review my payment flow for unsafe client-side trust or leaked secrets.
```

```text
Design payment database tables for attempts, idempotency keys, provider IDs, webhook events, refunds, and audit logs.
```

```text
What should happen if the app loses internet after the user completes payment?
```

```text
Design payment statuses for created, pending, requires_action, paid, failed, refunded, and cancelled states.
```

## Extra Socket And Realtime Starters

```text
Explain when to use Laravel Reverb/Echo versus Socket.IO.
```

```text
Help me configure WebSocket URLs for browser, Android emulator, physical device, and production wss.
```

```text
My private Laravel Echo channel auth fails. Help me check session, token, CSRF, and routes/channels.php.
```

```text
Help me run Laravel Reverb behind Nginx with WebSocket upgrade headers.
```

## PHP And Composer Starters

```text
Composer install/update is failing. Help me diagnose package, PHP version, extension, and SSL issues.
```

```text
Explain composer install vs composer update for Laravel applications.
```

```text
Help me fix PHP class not found, namespace, autoload, or missing extension errors.
```

```text
Review this PHP/Laravel code for type safety, exceptions, and maintainability.
```

```text
Help me configure required PHP extensions for a Laravel project.
```

## Frontend, Blade, And Vite Starters

```text
My Laravel Vite assets are not loading. Help me debug npm, @vite, manifest, and build issues.
```

```text
My Blade form does not submit correctly. Help me check method, CSRF, old input, validation errors, and route.
```

```text
My JavaScript fetch request fails. Help me inspect headers, CSRF, JSON, status codes, and API response body.
```

```text
Help me improve frontend performance for a Laravel/NativePHP screen.
```

## Git And Workflow Starters

```text
Help me create a safe Git workflow for this Laravel feature branch.
```

```text
I have a Git merge conflict. Explain how to resolve it safely.
```

```text
Review my staged changes and suggest a clear commit message.
```

```text
Create a .gitignore checklist for Laravel/NativePHP secrets and generated files.
```

## Testing And Code Quality Starters

```text
Create Laravel feature tests for this route/controller/database behavior.
```

```text
Help me test a NativePHP sync, payment, or API flow safely.
```

```text
Review this code for bugs, missing validation, missing authorization, performance issues, and missing tests.
```

```text
Help me use factories, RefreshDatabase, Queue::fake, Mail::fake, and Notification::fake in tests.
```

## Architecture Starters

```text
Should this Laravel logic go in a controller, service, action, job, event, listener, or model?
```

```text
Review my feature architecture and tell me if repositories, DTOs, services, or actions are actually needed.
```

```text
Design a maintainable architecture for a NativePHP app with offline sync and remote API.
```

```text
Identify anti-patterns in this Laravel/NativePHP code.
```

## Integration Starters

```text
Help me integrate an external API with timeout, retries, logging, and queue jobs.
```

```text
Design webhook storage and idempotent processing for an external provider.
```

```text
Help me build a large CSV/XLSX import flow with validation, row errors, and queued processing.
```

```text
Create an email/SMS/OTP delivery flow with rate limits, expiry, attempts, and provider logging.
```

## Storage, Files, Mail, And Queues Starters

```text
My Laravel email is not sending. Help me check mail config, queues, SMTP, and provider logs.
```

```text
Help me protect private file downloads with signed URLs.
```

```text
My Laravel queue is not processing jobs. Help me debug workers, failed jobs, connection, and old code.
```

```text
Design a queue job with retries, timeout, idempotency, and failure handling.
```

```text
Help me design safe file upload storage for public files, private files, and NativePHP mobile uploads.
```

## Extra Deployment And Production Starters

```text
Help me configure Laravel scheduler, queue workers, logs, backups, and monitoring for production.
```

```text
Create a production readiness checklist for Laravel API plus NativePHP mobile app.
```

```text
Help me safely run Laravel migrations during deployment.
```

```text
My production queue backlog is growing. Help me triage workers, failed jobs, timeouts, and retries.
```

```text
My database is slow or overloaded. Give me a safe triage checklist.
```

## Emulator And ADB Starters

```text
My Android emulator black screen or GPU issue happens. Help me choose the right emulator flags.
```

```text
Help me configure ANDROID_HOME, ANDROID_SDK_ROOT, platform-tools, and emulator path on Windows.
```

```text
Explain 10.0.2.2, localhost, LAN IP, and adb reverse for Android development.
```

```text
Help me read adb logcat for a NativePHP Android crash.
```

```text
My NativePHP Android app works on emulator but not on physical device. Help me debug networking.
```

## Extra Security Starters

```text
Review my app for SQL injection, XSS, CSRF, token leakage, and insecure payment handling.
```

```text
Help me respond safely if an API key, database password, or payment secret was leaked.
```

```text
What data should be encrypted or avoided in NativePHP local storage?
```

```text
Review this Laravel policy/gate setup for authorization mistakes.
```

## GPT Setup Starters

```text
Help me improve this custom GPT instruction so it answers Laravel/NativePHP problems more accurately.
```

```text
Turn this bug pattern into training knowledge for my Laravel/NativePHP GPT.
```

```text
Create a concise checklist-style knowledge note from this troubleshooting conversation.
```

```text
Review my GPT knowledge files and tell me what important Laravel/NativePHP topics are missing.
```
