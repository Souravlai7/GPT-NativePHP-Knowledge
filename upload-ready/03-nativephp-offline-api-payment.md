# nativephp offline api payment


---

# Source: nativephp-offline-api-payment.md

# NativePHP Offline, API, Local DB, Sync, And Payment Knowledge

Use this file as advanced NativePHP/mobile training data for local database storage, offline-first workflows, API calling, internet connection checks, sync logic, payments, and production-safe architecture.

## Core Mobile Architecture Rule

For production mobile apps:

```text
NativePHP app -> local storage/cache -> HTTPS Laravel API -> backend database/payment provider
```

Avoid:

```text
NativePHP app -> direct production MySQL connection
```

Reason:

- APK/IPA can be inspected.
- Database credentials can leak.
- Direct DB access bypasses API validation and authorization.
- Offline/retry/sync needs app-level logic.
- Payments and sensitive actions need server-side verification.

## Local Database Options

Common local storage choices:

```text
SQLite database
Laravel cache
Laravel filesystem storage
Encrypted key/value storage through native plugin if available
Browser/webview localStorage or IndexedDB for frontend-only state
```

Use SQLite for:

- Offline records.
- Draft forms.
- Cached API responses.
- Sync queues.
- Local app settings.
- Audit/history that must persist.

Use secure native storage for:

- Auth tokens.
- Refresh tokens.
- Sensitive small secrets.

Do not store sensitive secrets in plain localStorage if avoidable.

## SQLite Local DB Pattern

Example `.env` for local SQLite:

```env
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
```

Create file:

```bash
type nul > database\database.sqlite
php artisan migrate
```

Linux/macOS:

```bash
touch database/database.sqlite
php artisan migrate
```

Migration example for offline records:

```php
Schema::create('offline_posts', function (Blueprint $table) {
    $table->id();
    $table->uuid('local_uuid')->unique();
    $table->unsignedBigInteger('remote_id')->nullable()->index();
    $table->string('title');
    $table->text('body')->nullable();
    $table->string('sync_status')->default('pending')->index();
    $table->text('sync_error')->nullable();
    $table->timestamp('last_synced_at')->nullable();
    $table->timestamps();
    $table->softDeletes();
});
```

Model casts:

```php
class OfflinePost extends Model
{
    protected $fillable = [
        'local_uuid',
        'remote_id',
        'title',
        'body',
        'sync_status',
        'sync_error',
        'last_synced_at',
    ];

    protected $casts = [
        'last_synced_at' => 'datetime',
    ];
}
```

## Sync Status Design

Recommended statuses:

```text
draft
pending_create
pending_update
pending_delete
syncing
synced
failed
conflict
```

Fields:

```text
local_uuid: stable local ID before server ID exists
remote_id: server ID after successful sync
sync_status: current sync state
sync_error: last sync error
last_synced_at: latest successful sync
server_updated_at: server version timestamp
version: integer optimistic locking version if used
deleted_at: soft delete for offline delete sync
```

Rules:

- Create local record immediately for good UX.
- Mark record `pending_create` or `pending_update`.
- Sync when internet is available.
- Store server ID after successful create.
- Do not delete local record permanently until delete sync succeeds.
- Keep sync operations idempotent.

## Offline Create Flow

Flow:

```text
User submits form
Validate locally
Save to SQLite with local_uuid
Mark sync_status=pending_create
Show record in UI immediately
If online, try sync
If sync succeeds, save remote_id and mark synced
If sync fails, keep pending/failed and show retry
```

Laravel local save example:

```php
$post = OfflinePost::create([
    'local_uuid' => (string) Str::uuid(),
    'title' => $validated['title'],
    'body' => $validated['body'] ?? null,
    'sync_status' => 'pending_create',
]);
```

Sync create request:

```php
$response = Http::timeout(15)
    ->acceptJson()
    ->withToken($token)
    ->post(config('services.api.url') . '/api/posts', [
        'client_uuid' => $post->local_uuid,
        'title' => $post->title,
        'body' => $post->body,
    ]);
```

On success:

```php
$post->update([
    'remote_id' => $response->json('data.id'),
    'sync_status' => 'synced',
    'sync_error' => null,
    'last_synced_at' => now(),
]);
```

On failure:

```php
$post->update([
    'sync_status' => 'failed',
    'sync_error' => $response->body(),
]);
```

## Idempotent API Design For Sync

The backend should accept a client-generated UUID:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'client_uuid' => ['required', 'uuid'],
        'title' => ['required', 'string', 'max:255'],
        'body' => ['nullable', 'string'],
    ]);

    $post = Post::firstOrCreate(
        [
            'user_id' => $request->user()->id,
            'client_uuid' => $validated['client_uuid'],
        ],
        [
            'title' => $validated['title'],
            'body' => $validated['body'] ?? null,
        ],
    );

    return new PostResource($post);
}
```

Why:

- If mobile retries after timeout, server does not create duplicate record.
- Client can safely retry `pending_create`.

Database:

```php
$table->uuid('client_uuid');
$table->foreignId('user_id')->constrained();
$table->unique(['user_id', 'client_uuid']);
```

## Offline Update Flow

Flow:

```text
User edits local record
Save locally
If record has remote_id, mark pending_update
If record is not synced yet, keep pending_create
Sync when online
```

Patch request:

```php
$response = Http::timeout(15)
    ->withToken($token)
    ->patch(config('services.api.url') . "/api/posts/{$post->remote_id}", [
        'title' => $post->title,
        'body' => $post->body,
        'version' => $post->version,
    ]);
```

Backend should validate ownership and version if conflict detection is needed.

## Offline Delete Flow

Do not immediately hard-delete synced records while offline.

Flow:

```text
If never synced: delete locally.
If synced: mark pending_delete and hide from UI.
When online: send DELETE to API.
If success: hard-delete locally or mark synced deleted.
```

Example:

```php
if ($post->remote_id === null) {
    $post->delete();
} else {
    $post->update(['sync_status' => 'pending_delete']);
}
```

Sync:

```php
Http::withToken($token)
    ->delete(config('services.api.url') . "/api/posts/{$post->remote_id}");
```

## Sync Queue Table

For complex apps, create a dedicated sync operations table:

```php
Schema::create('sync_operations', function (Blueprint $table) {
    $table->id();
    $table->uuid('operation_uuid')->unique();
    $table->string('model_type');
    $table->string('local_uuid')->nullable();
    $table->unsignedBigInteger('remote_id')->nullable();
    $table->string('operation'); // create, update, delete
    $table->json('payload')->nullable();
    $table->string('status')->default('pending')->index();
    $table->unsignedInteger('attempts')->default(0);
    $table->text('last_error')->nullable();
    $table->timestamp('next_retry_at')->nullable();
    $table->timestamps();
});
```

Retry rules:

```text
attempt 1: immediately
attempt 2: after 30 seconds
attempt 3: after 2 minutes
attempt 4: after 10 minutes
after too many failures: mark failed and show user
```

## Conflict Handling

Conflict happens when local and server records both changed.

Strategies:

```text
Client wins: local overwrites server.
Server wins: server overwrites local.
Last write wins: newest timestamp wins.
Manual merge: user chooses.
Field-level merge: non-overlapping fields merge automatically.
```

For important business data, avoid silent overwrites.

Backend optimistic lock:

```php
$validated = $request->validate([
    'title' => ['required', 'string'],
    'version' => ['required', 'integer'],
]);

if ($post->version !== $validated['version']) {
    return response()->json([
        'message' => 'Conflict detected.',
        'data' => new PostResource($post),
    ], 409);
}

$post->update([
    'title' => $validated['title'],
    'version' => $post->version + 1,
]);
```

Client on 409:

```text
Mark record conflict.
Show server version and local version.
Let user choose or merge.
```

## Internet Connection Checks

Do not trust one signal only. A device can be connected to Wi-Fi but have no internet.

Possible checks:

- Native network status plugin if available.
- Lightweight API health endpoint.
- Timeout-based HTTP request.
- Retry failed requests.

Backend health endpoint:

```php
Route::get('/health', function () {
    return response()->json([
        'ok' => true,
        'time' => now()->toISOString(),
    ]);
});
```

PHP HTTP connectivity check:

```php
try {
    $response = Http::timeout(5)
        ->acceptJson()
        ->get(config('services.api.url') . '/health');

    $online = $response->ok();
} catch (Throwable $e) {
    $online = false;
}
```

JavaScript browser/webview check:

```js
async function isOnline() {
  try {
    const response = await fetch(`${apiBaseUrl}/health`, {
      method: 'GET',
      cache: 'no-store',
    });

    return response.ok;
  } catch {
    return false;
  }
}
```

Browser signal:

```js
window.addEventListener('online', () => {
  console.log('browser says online');
});

window.addEventListener('offline', () => {
  console.log('browser says offline');
});
```

Warning:

```text
navigator.onLine can be wrong. Confirm with an API health request before syncing important data.
```

## API Client Pattern

Centralize API calls instead of scattering raw requests.

PHP service:

```php
class ApiClient
{
    public function __construct(private string $baseUrl, private ?string $token = null)
    {
    }

    public function get(string $path): Response
    {
        return Http::timeout(15)
            ->retry(2, 300)
            ->acceptJson()
            ->when($this->token, fn ($http) => $http->withToken($this->token))
            ->get($this->baseUrl . $path);
    }

    public function post(string $path, array $payload): Response
    {
        return Http::timeout(15)
            ->retry(2, 300)
            ->acceptJson()
            ->when($this->token, fn ($http) => $http->withToken($this->token))
            ->post($this->baseUrl . $path, $payload);
    }
}
```

JavaScript API wrapper:

```js
async function apiRequest(path, options = {}) {
  const response = await fetch(`${apiBaseUrl}${path}`, {
    ...options,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...(options.headers || {}),
    },
  });

  const text = await response.text();
  const data = text ? JSON.parse(text) : null;

  if (!response.ok) {
    const error = new Error(data?.message || 'API request failed');
    error.status = response.status;
    error.data = data;
    throw error;
  }

  return data;
}
```

Rules:

- Set timeout.
- Parse error response.
- Handle 401 by logout/token refresh.
- Handle 422 by showing validation errors.
- Handle 409 by conflict UI.
- Handle 429 by backoff.
- Handle 500 by retry later or show support message.

## API Status Handling

```text
200: success
201: created
204: success with no body
401: token/session invalid, login again
403: user lacks permission
404: missing record or wrong endpoint
409: conflict during sync
422: validation error
429: rate limited, retry later
500: server error
503: service unavailable
```

## Auth Token Storage

Rules:

- Prefer secure storage plugin for tokens.
- Avoid plain localStorage for sensitive tokens if better storage exists.
- Clear token on logout.
- Revoke token on backend if possible.
- Do not log tokens.

Token table on backend:

```text
user_id
device_name
token
last_used_at
revoked_at
```

Logout:

```php
$request->user()->currentAccessToken()?->delete();
```

## Payments In NativePHP

Production-safe payment architecture:

```text
NativePHP app -> Laravel API creates payment intent/order -> payment provider SDK/page -> provider webhook -> Laravel verifies -> app displays status
```

Never:

- Trust client-only payment success.
- Store card data in app.
- Put secret payment keys in app.
- Mark order paid only because frontend says success.

Payment flow:

```text
1. User starts checkout in app.
2. App calls Laravel API: create order/payment intent.
3. Backend creates payment intent with provider secret key.
4. App completes payment using provider-approved client flow.
5. Provider sends webhook to backend.
6. Backend verifies webhook signature.
7. Backend marks payment paid/failed.
8. App polls or listens for updated payment status.
```

Payment statuses:

```text
created
pending
requires_action
authorized
captured
paid
failed
cancelled
refunded
partially_refunded
```

Backend migration fields:

```php
Schema::create('payments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->foreignId('order_id')->constrained();
    $table->string('provider');
    $table->string('provider_payment_id')->nullable()->index();
    $table->string('status')->index();
    $table->decimal('amount', 10, 2);
    $table->string('currency', 3);
    $table->uuid('idempotency_key')->unique();
    $table->json('provider_payload')->nullable();
    $table->timestamps();
});
```

Create payment endpoint:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'order_id' => ['required', 'exists:orders,id'],
        'idempotency_key' => ['required', 'uuid'],
    ]);

    $payment = Payment::firstOrCreate(
        ['idempotency_key' => $validated['idempotency_key']],
        [
            'user_id' => $request->user()->id,
            'order_id' => $validated['order_id'],
            'provider' => 'provider_name',
            'status' => 'created',
            'amount' => $order->total,
            'currency' => 'INR',
        ],
    );

    return response()->json([
        'data' => $payment,
    ], 201);
}
```

Webhook rules:

- Verify signature.
- Store event ID.
- Process idempotently.
- Do not process duplicate webhook twice.
- Return 2xx after accepting event.
- Keep raw payload if safe.

Webhook event table:

```php
Schema::create('payment_webhook_events', function (Blueprint $table) {
    $table->id();
    $table->string('provider');
    $table->string('event_id')->unique();
    $table->string('event_type');
    $table->json('payload');
    $table->timestamp('processed_at')->nullable();
    $table->timestamps();
});
```

Payment sync in app:

```text
After payment attempt, app should refresh order/payment status from backend.
If app loses internet after payment, do not assume failure. Show pending and sync later.
If backend webhook later marks paid, app updates status on next sync.
```

## Offline Payments

Most real payments require internet.

Offline payment rules:

- Do not mark payment paid offline.
- Allow creating a local pending order if business allows.
- Sync order when online.
- Start payment only when online.
- If payment app/provider opens and result is unclear, verify with backend/provider.

Local status:

```text
payment_status=pending_sync
payment_status=pending_payment
payment_status=payment_processing
payment_status=paid
payment_status=failed
```

## Sync With Payments

Danger:

```text
Duplicate payment attempts can charge users twice.
```

Rules:

- Use idempotency keys.
- Store payment attempt locally.
- Disable button while request is in progress.
- On timeout, check backend payment status before creating a new payment.
- Reconcile with webhook.

Local payment attempt fields:

```text
local_uuid
remote_payment_id
order_local_uuid
order_remote_id
idempotency_key
status
amount
currency
last_checked_at
error
```

## Background Sync

Sync triggers:

- App starts.
- User logs in.
- Internet becomes available.
- User taps Sync.
- App resumes from background.
- Periodic timer while app is open.

Do not assume background timers run forever on mobile.

Sync order:

```text
1. Auth/session check.
2. Push local creates.
3. Push local updates.
4. Push local deletes.
5. Pull server changes.
6. Resolve conflicts.
7. Update last sync timestamp.
```

Sync lock:

```php
if (Cache::get('sync_running')) {
    return;
}

Cache::put('sync_running', true, now()->addMinutes(5));

try {
    // sync
} finally {
    Cache::forget('sync_running');
}
```

## Pull Changes From Server

Backend endpoint:

```php
Route::get('/sync/posts', [PostSyncController::class, 'index']);
```

Request:

```text
GET /api/sync/posts?updated_after=2026-05-01T10:00:00Z
```

Response:

```json
{
  "data": [],
  "server_time": "2026-05-04T10:00:00Z"
}
```

Rules:

- Use server timestamps for sync cursor.
- Include deleted records if clients must delete locally.
- Paginate large sync responses.

## Data Storage Safety

Local data rules:

- Encrypt sensitive data if possible.
- Minimize stored personal data.
- Clear local data on logout when required.
- Keep cache expiration policy.
- Add export/backup if local-only data matters.
- Handle app update migrations carefully.

Do not store:

- Payment secret keys.
- Raw card numbers.
- CVV.
- Database root password.
- Admin API keys.

## GPT Response Rules For These Topics

When user asks about NativePHP local DB:

```text
Ask whether data is local-only, offline-first sync, or remote backend cache. Recommend SQLite for local records and API backend for production shared data. Warn against direct production MySQL credentials in the app.
```

When user asks about API calling:

```text
Give base URL rules, timeout/error handling, token handling, and HTTP status handling. For Android emulator, mention 10.0.2.2.
```

When user asks about sync:

```text
Explain local_uuid, remote_id, sync_status, retry/backoff, idempotent create, conflict handling, and health check.
```

When user asks about internet check:

```text
Say navigator.onLine is not enough. Use a lightweight /health endpoint with timeout.
```

When user asks about payment:

```text
Secret keys and final payment verification must stay on backend. Use idempotency keys, provider webhooks, and backend status refresh. Never mark paid only from client result.
```

