# architecture testing git


---

# Source: architecture-patterns.md

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


---

# Source: testing-quality-knowledge.md

# Testing And Code Quality Knowledge

Use this file as testing, refactoring, review, and quality training/reference data for Laravel/PHP projects.

## Testing Strategy

Use different tests for different risk:

```text
Unit tests: small pure logic.
Feature tests: HTTP routes, controllers, database behavior.
Integration tests: multiple services working together.
Browser tests: critical UI flows when needed.
```

Most Laravel business features should have feature tests because they verify routing, middleware, validation, database writes, and responses together.

## Run Tests

```bash
php artisan test
php artisan test --filter=UserTest
php artisan test --parallel
./vendor/bin/phpunit
```

## Feature Test Example

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class PostTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_post(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/posts', [
            'title' => 'My post',
            'body' => 'Post body',
        ]);

        $response->assertRedirect();

        $this->assertDatabaseHas('posts', [
            'title' => 'My post',
            'user_id' => $user->id,
        ]);
    }
}
```

## API Test Example

```php
public function test_api_returns_posts(): void
{
    Post::factory()->count(3)->create();

    $this->getJson('/api/posts')
        ->assertOk()
        ->assertJsonStructure([
            'data' => [
                ['id', 'title'],
            ],
        ]);
}
```

## Validation Test Example

```php
public function test_title_is_required(): void
{
    $user = User::factory()->create();

    $this->actingAs($user)
        ->post('/posts', ['title' => ''])
        ->assertSessionHasErrors('title');
}
```

## Useful Assertions

```php
assertOk()
assertCreated()
assertNoContent()
assertRedirect()
assertForbidden()
assertNotFound()
assertStatus(422)
assertJson()
assertJsonPath()
assertJsonValidationErrors()
assertSessionHasErrors()
assertDatabaseHas()
assertDatabaseMissing()
```

## Factories

```php
User::factory()->create();
User::factory()->count(10)->create();
Post::factory()->for($user)->create();
```

Factory state:

```php
public function published(): static
{
    return $this->state(fn () => [
        'status' => 'published',
        'published_at' => now(),
    ]);
}
```

Usage:

```php
Post::factory()->published()->create();
```

## Code Review Checklist

- Does the code solve the requested behavior?
- Are edge cases handled?
- Is validation present?
- Is authorization present?
- Are database queries efficient?
- Are transactions needed?
- Are errors logged or surfaced correctly?
- Are secrets protected?
- Are tests added or updated?
- Is the code consistent with existing project patterns?

## Refactoring Rules

- Refactor only with a clear reason.
- Keep behavior unchanged unless intentionally fixing behavior.
- Add tests before risky refactors.
- Prefer small functions with clear names.
- Remove duplication when it causes maintenance risk.
- Avoid adding abstractions that only hide simple code.

## Common Test Problems

Database not reset:

```php
use RefreshDatabase;
```

Auth missing:

```php
$this->actingAs($user);
```

CSRF in tests:

```text
Laravel disables CSRF middleware in tests by default for typical feature tests.
```

Factories missing required fields:

```text
Update factory definition when migration/model changes.
```

Queue jobs during tests:

```php
Queue::fake();
Queue::assertPushed(ProcessOrder::class);
```

Mail during tests:

```php
Mail::fake();
Mail::assertSent(WelcomeMail::class);
```

Notifications:

```php
Notification::fake();
Notification::assertSentTo($user, InvoicePaid::class);
```


---

# Source: git-workflow-knowledge.md

# Git Workflow Knowledge

Use this file as Git and collaboration training/reference data.

## Basics

Check status:

```bash
git status
```

See changes:

```bash
git diff
git diff --staged
```

Stage and commit:

```bash
git add file.php
git commit -m "Fix user import validation"
```

History:

```bash
git log --oneline --decorate --graph
git show HEAD
```

Branches:

```bash
git branch
git switch -c feature/nativephp-login
git switch main
```

Remote:

```bash
git fetch
git pull
git push
git push -u origin feature/nativephp-login
```

## Safe Daily Workflow

```bash
git status
git fetch
git switch main
git pull
git switch -c feature/task-name
```

Before committing:

```bash
git status
git diff
php artisan test
```

## Merge Conflicts

Conflict markers:

```text
<<<<<<< HEAD
current branch changes
=======
incoming changes
>>>>>>> branch-name
```

Resolve by editing the file to the desired final content, then:

```bash
git add conflicted-file.php
git commit
```

## Useful Commands

Undo unstaged file change:

```bash
git restore file.php
```

Unstage:

```bash
git restore --staged file.php
```

Show who changed a line:

```bash
git blame file.php
```

Search history:

```bash
git log --grep="payment"
git log -S "oldFunctionName"
```

Stash:

```bash
git stash
git stash pop
git stash list
```

## Commit Message Style

Good messages:

```text
Fix checkout validation for zero amount orders
Add NativePHP Android emulator troubleshooting notes
Improve user import duplicate email handling
```

Avoid vague messages:

```text
fix
changes
update
final
```

## Git Ignore

Usually ignored:

```text
/vendor
/node_modules
/.env
/storage/logs/*.log
/public/build
```

Usually committed:

```text
composer.json
composer.lock
package.json
package-lock.json
database/migrations
config files without secrets
```

