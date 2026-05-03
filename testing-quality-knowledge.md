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
