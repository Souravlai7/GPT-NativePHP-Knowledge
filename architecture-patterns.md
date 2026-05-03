# Architecture And Design Patterns

Use this file to help GPT make better Laravel architecture decisions for real applications.

## Core Principle

Start simple. Add structure only when complexity exists.

Good default:

```text
Route -> Controller -> Form Request -> Model/Service -> Database -> Response
```

Do not create repositories, interfaces, actions, DTOs, and service layers for every small CRUD feature unless the project already uses that pattern or the logic is complex.

## Thin Controllers

Controllers should usually:

- Receive request.
- Authorize action.
- Validate input.
- Call model/service/action.
- Return response.

Example:

```php
public function store(StoreInvoiceRequest $request, InvoiceService $service)
{
    $this->authorize('create', Invoice::class);

    $invoice = $service->create($request->user(), $request->validated());

    return redirect()->route('invoices.show', $invoice);
}
```

## Service Classes

Use services when:

- Logic is reused in multiple places.
- Logic has multiple steps.
- External APIs are involved.
- Transactions are needed.
- Controller becomes hard to read.

Example:

```php
class InvoiceService
{
    public function create(User $user, array $data): Invoice
    {
        return DB::transaction(function () use ($user, $data) {
            $invoice = $user->invoices()->create($data);
            InvoiceCreated::dispatch($invoice);

            return $invoice;
        });
    }
}
```

## Action Classes

Use action classes for one focused business operation:

```php
class ApproveLeaveRequest
{
    public function handle(LeaveRequest $leave, User $approver): void
    {
        if (! $approver->can('approve', $leave)) {
            abort(403);
        }

        $leave->update(['status' => 'approved']);
    }
}
```

Good for:

- `CreateOrder`
- `ApproveLeaveRequest`
- `ImportUsers`
- `CancelSubscription`

## Form Requests

Use form requests when validation is more than a few simple rules or reused:

```php
class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', User::class);
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
        ];
    }
}
```

## DTOs

Use DTOs when:

- Data crosses boundaries.
- Many arrays become unclear.
- External API payloads need structure.

Simple DTO:

```php
readonly class CreateUserData
{
    public function __construct(
        public string $name,
        public string $email,
    ) {
    }
}
```

Avoid DTOs for very small one-controller CRUD unless the project already uses them.

## Repository Pattern

Laravel Eloquent already works like a repository for many apps.

Use repositories when:

- Storage implementation may change.
- Queries are complex and reused.
- Project already follows repository pattern.

Avoid repositories when they only wrap Eloquent one-line methods:

```php
public function find($id) {
    return User::find($id);
}
```

## Events And Jobs

Use events for "something happened":

```php
OrderPlaced::dispatch($order);
```

Use jobs for work that can happen later:

```php
SendOrderEmail::dispatch($order);
```

Good queued work:

- Email sending.
- PDF generation.
- Image processing.
- External API sync.
- Imports/exports.
- Notifications.

Keep synchronous:

- Required validation.
- Required payment authorization response.
- Immediate database state required for response.

## Transactions

Use transactions when multiple writes must succeed together:

```php
DB::transaction(function () use ($order, $paymentData) {
    $payment = Payment::create($paymentData);
    $order->update(['payment_id' => $payment->id, 'status' => 'paid']);
});
```

Do not do slow external API calls inside transactions when avoidable.

## API Architecture

Use API resources for consistent output:

```php
return new UserResource($user);
return UserResource::collection($users);
```

Keep response shape stable:

```json
{
  "data": {},
  "message": "Optional human message"
}
```

Use pagination metadata for lists.

## NativePHP Architecture

For NativePHP/mobile:

- Keep Laravel app logic independent from native shell where possible.
- Put device-specific behavior behind services or NativePHP plugins.
- Do not hardcode emulator URLs in production.
- Prefer backend APIs for production mobile apps.
- Do not store production DB credentials in the app.
- Design offline behavior intentionally if needed.

## Anti-Patterns

- Fat controllers with validation, SQL, API calls, and business rules mixed together.
- Querying database inside Blade loops.
- Using `env()` directly outside config files.
- Large service class with unrelated methods.
- Catching all exceptions and returning success.
- Running slow imports in HTTP requests.
- Adding abstractions before there is complexity.
- Storing secrets in Git.
