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
