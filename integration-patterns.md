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
