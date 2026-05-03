# Performance And Scalability

Use this file to help GPT solve real performance problems in Laravel, MySQL, APIs, queues, and frontend apps.

## Performance Rule

Measure first, optimize second.

Useful data:

- Slow URL.
- Request duration.
- Query count.
- Slowest query.
- Memory usage.
- Payload size.
- Queue time.
- Cache hit/miss.
- Traffic volume.

## Laravel Performance Checklist

- Enable config, route, and view cache in production.
- Avoid debug tools in production.
- Avoid N+1 queries.
- Paginate large lists.
- Select only needed columns.
- Cache expensive stable data.
- Queue slow tasks.
- Use indexes for frequent filters.
- Avoid heavy work in Blade views.
- Avoid repeated filesystem/network calls during requests.

Production cache:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan optimize
```

## N+1 Query Fix

Bad:

```php
$orders = Order::latest()->get();

foreach ($orders as $order) {
    echo $order->user->name;
}
```

Good:

```php
$orders = Order::with('user')->latest()->paginate(25);
```

## Large Data Processing

Bad:

```php
User::all()->each(fn ($user) => $this->process($user));
```

Good:

```php
User::chunkById(1000, function ($users) {
    foreach ($users as $user) {
        ProcessUser::dispatch($user->id);
    }
});
```

Cursor:

```php
foreach (User::where('active', true)->cursor() as $user) {
    //
}
```

## Caching Patterns

Remember:

```php
$settings = Cache::remember('settings', 3600, function () {
    return Setting::pluck('value', 'key');
});
```

Forget after update:

```php
Cache::forget('settings');
```

Cache key with model ID:

```php
Cache::remember("user:{$user->id}:stats", 600, fn () => $this->statsFor($user));
```

Rules:

- Cache data that is expensive and read often.
- Invalidate cache when source data changes.
- Do not cache user-private data under shared keys.
- Use short TTL for frequently changing dashboards.

## API Performance

- Use pagination.
- Avoid returning huge nested relationships.
- Use API resources.
- Compress responses at web server/CDN level.
- Add rate limits.
- Cache public read-heavy endpoints.
- Use `ETag` or `Last-Modified` when appropriate.
- Move exports to background jobs.

## Frontend Performance

- Build production assets.
- Avoid loading unused large JS libraries.
- Compress and resize images.
- Use lazy loading for below-fold images.
- Avoid rendering thousands of DOM nodes.
- Debounce search inputs.
- Show loading states for slow API calls.

Debounce idea:

```js
let timer;
input.addEventListener('input', () => {
  clearTimeout(timer);
  timer = setTimeout(search, 300);
});
```

## Queue Scaling

Use queues for:

- Emails.
- Imports.
- Exports.
- Reports.
- Image processing.
- Webhook processing.
- Slow third-party API calls.

Worker tuning:

```bash
php artisan queue:work --queue=high,default --tries=3 --timeout=120
```

Rules:

- Keep jobs idempotent when possible.
- Store external IDs.
- Use retries with backoff.
- Do not retry permanent validation errors forever.
- Monitor failed jobs.

## Database Scaling

Before scaling hardware:

- Add missing indexes.
- Reduce query count.
- Avoid full table scans.
- Archive old data.
- Cache read-heavy data.
- Move reports off request path.

Read/write splitting can help read-heavy systems but adds replication lag complexity.

## NativePHP/Mobile Performance

- Avoid huge initial payloads.
- Cache locally where safe.
- Compress images.
- Avoid blocking UI while waiting for network.
- Handle offline and retry states.
- Do not make many small sequential API calls on app start.
- Bundle production assets for release builds.
