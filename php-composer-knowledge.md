# PHP And Composer Knowledge

Use this file as PHP and Composer training/reference data for Laravel projects.

## PHP Basics

Variables:

```php
$name = 'Laravel';
$count = 10;
$active = true;
```

Arrays:

```php
$items = ['one', 'two'];
$user = ['name' => 'Amit', 'email' => 'amit@example.com'];
```

Null coalescing:

```php
$name = $data['name'] ?? 'Guest';
```

Strict comparison:

```php
if ($status === 'active') {
    //
}
```

Functions:

```php
function fullName(string $first, string $last): string
{
    return trim($first . ' ' . $last);
}
```

Classes:

```php
class InvoiceService
{
    public function total(array $items): float
    {
        return array_sum(array_column($items, 'amount'));
    }
}
```

Constructor property promotion:

```php
class SendInvoice
{
    public function __construct(
        private Mailer $mailer,
        private InvoiceRepository $invoices,
    ) {
    }
}
```

Enums:

```php
enum OrderStatus: string
{
    case Draft = 'draft';
    case Paid = 'paid';
    case Cancelled = 'cancelled';
}
```

## PHP Error Handling

Throw exception:

```php
throw new RuntimeException('Invalid payment state.');
```

Try/catch:

```php
try {
    $service->charge($order);
} catch (Throwable $e) {
    report($e);
    return back()->withErrors('Payment failed.');
}
```

Laravel helpers:

```php
abort(404);
abort_if(! $user->active, 403);
throw_if($amount <= 0, InvalidArgumentException::class);
```

## Composer Commands

Install dependencies:

```bash
composer install
```

Add package:

```bash
composer require vendor/package
```

Add dev package:

```bash
composer require --dev vendor/package
```

Update dependencies:

```bash
composer update
composer update vendor/package
```

Autoload refresh:

```bash
composer dump-autoload
composer dump-autoload -o
```

Show packages:

```bash
composer show
composer show vendor/package
```

Diagnose:

```bash
composer diagnose
composer why vendor/package
composer why-not vendor/package 2.0
```

## composer.json Concepts

Important sections:

```json
{
  "require": {
    "php": "^8.2",
    "laravel/framework": "^11.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^11.0"
  },
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

Rules:

- Commit `composer.json`.
- Commit `composer.lock` for applications.
- Do not commit `vendor/`.
- Run `composer install` in production, not broad `composer update`.
- Use `composer update` intentionally during dependency upgrades.

## PHP Quality

Static analysis:

```bash
./vendor/bin/phpstan analyse
./vendor/bin/pint
./vendor/bin/phpunit
```

Common coding rules:

- Prefer dependency injection over global state.
- Keep controllers thin.
- Move business rules into services, actions, jobs, or domain classes.
- Validate input at boundaries.
- Use typed method parameters and return types.
- Avoid catching exceptions only to hide them.
- Do not store secrets in code.

## Common PHP Problems

Class not found:

```bash
composer dump-autoload
php artisan optimize:clear
```

Memory limit:

```ini
memory_limit=512M
```

Max execution time:

```ini
max_execution_time=120
```

Extension missing:

```bash
php -m
php --ini
```

Common extensions for Laravel:

```text
openssl
pdo
pdo_mysql
mbstring
tokenizer
xml
ctype
json
curl
fileinfo
zip
```
