# laravel core


---

# Source: laravel-cheatsheet.md

# Laravel Cheatsheet

Use this file as Laravel training/reference data for common commands, project structure, debugging, Eloquent, routing, validation, queues, cache, jobs, events, APIs, testing, and deployment.

## Project Setup

Create a project:

```bash
composer create-project laravel/laravel app-name
cd app-name
php artisan serve
```

Install dependencies:

```bash
composer install
npm install
npm run dev
```

Copy environment:

```bash
cp .env.example .env
php artisan key:generate
```

Common `.env` values:

```env
APP_NAME=Laravel
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=root
DB_PASSWORD=

CACHE_STORE=file
QUEUE_CONNECTION=database
SESSION_DRIVER=file
```

## Important Artisan Commands

Serve app:

```bash
php artisan serve
php artisan serve --host=0.0.0.0 --port=8000
```

List commands:

```bash
php artisan list
php artisan help migrate
```

Clear generated/cache files:

```bash
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear
php artisan event:clear
```

Production optimization:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache
php artisan optimize
```

Maintenance mode:

```bash
php artisan down
php artisan down --secret="deploy-secret"
php artisan up
```

Tinker:

```bash
php artisan tinker
User::count();
```

## Make Commands

```bash
php artisan make:model Post
php artisan make:model Post -m
php artisan make:model Post -mcr
php artisan make:controller PostController
php artisan make:controller PostController --resource
php artisan make:request StorePostRequest
php artisan make:middleware CheckRole
php artisan make:policy PostPolicy --model=Post
php artisan make:migration create_posts_table
php artisan make:seeder UserSeeder
php artisan make:factory PostFactory
php artisan make:job SendWelcomeEmail
php artisan make:event OrderPlaced
php artisan make:listener SendOrderNotification
php artisan make:mail WelcomeMail
php artisan make:notification InvoicePaid
php artisan make:command ImportUsers
php artisan make:test PostTest
php artisan make:test PostTest --unit
```

## Directory Structure

Important folders:

```text
app/Http/Controllers       Web/API controllers
app/Http/Middleware        Request middleware
app/Http/Requests          Form request validation
app/Models                 Eloquent models
app/Policies               Authorization policies
app/Providers              Service providers
bootstrap/app.php          App bootstrap and routing config in newer Laravel versions
config/                    App/package config
database/migrations        Schema changes
database/factories         Model factories
database/seeders           Seed data
resources/views            Blade templates
resources/js               Frontend JavaScript
resources/css              Frontend CSS
routes/web.php             Web routes with sessions/cookies
routes/api.php             API routes
storage/logs/laravel.log   Laravel log file
tests/Feature              HTTP/application tests
tests/Unit                 Unit tests
```

## Routing

Basic routes:

```php
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/posts', [PostController::class, 'index']);
Route::post('/posts', [PostController::class, 'store']);
Route::get('/posts/{post}', [PostController::class, 'show']);
Route::put('/posts/{post}', [PostController::class, 'update']);
Route::delete('/posts/{post}', [PostController::class, 'destroy']);
```

Named routes:

```php
Route::get('/dashboard', DashboardController::class)->name('dashboard');

route('dashboard');
redirect()->route('dashboard');
```

Route groups:

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
});

Route::prefix('admin')->name('admin.')->middleware('auth')->group(function () {
    Route::resource('users', AdminUserController::class);
});
```

Resource routes:

```php
Route::resource('posts', PostController::class);
Route::apiResource('posts', PostApiController::class);
```

Route model binding:

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post;
});
```

Useful route commands:

```bash
php artisan route:list
php artisan route:list --name=posts
php artisan route:list --path=api
```

## Controllers

Single action controller:

```php
class DashboardController
{
    public function __invoke()
    {
        return view('dashboard');
    }
}
```

Resource controller methods:

```php
index()    list records
create()   show create form
store()    save new record
show()     show one record
edit()     show edit form
update()   update record
destroy()  delete record
```

## Requests And Validation

Inline validation:

```php
$validated = $request->validate([
    'name' => ['required', 'string', 'max:255'],
    'email' => ['required', 'email', 'unique:users,email'],
    'password' => ['required', 'string', 'min:8', 'confirmed'],
]);
```

Form request:

```php
class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body' => ['required', 'string'],
            'published_at' => ['nullable', 'date'],
        ];
    }
}
```

Controller usage:

```php
public function store(StorePostRequest $request)
{
    Post::create($request->validated());
}
```

Common validation rules:

```text
required, nullable, sometimes, string, integer, numeric, boolean, array
email, url, date, image, file, mimes:jpg,png,pdf, max:2048
min:3, max:255, between:1,10, in:draft,published
exists:users,id, unique:users,email
confirmed, accepted
```

## Eloquent Models

Model example:

```php
class Post extends Model
{
    protected $fillable = [
        'title',
        'body',
        'user_id',
        'published_at',
    ];

    protected $casts = [
        'published_at' => 'datetime',
        'is_featured' => 'boolean',
        'meta' => 'array',
    ];
}
```

Create/update/delete:

```php
Post::create($data);

$post->update($data);
$post->delete();

Post::where('is_published', true)->update(['status' => 'published']);
```

Queries:

```php
Post::query()->where('status', 'published')->get();
Post::where('user_id', $userId)->first();
Post::where('email', $email)->firstOrFail();
Post::latest()->paginate(15);
Post::orderBy('created_at', 'desc')->limit(10)->get();
```

Useful methods:

```php
first()
firstOrFail()
firstOrCreate()
updateOrCreate()
find()
findOrFail()
pluck()
value()
count()
exists()
doesntExist()
paginate()
simplePaginate()
cursorPaginate()
```

Scopes:

```php
public function scopePublished($query)
{
    return $query->where('status', 'published');
}

Post::published()->latest()->get();
```

Accessors/mutators:

```php
protected function title(): Attribute
{
    return Attribute::make(
        get: fn ($value) => ucfirst($value),
        set: fn ($value) => trim($value),
    );
}
```

## Relationships

One to many:

```php
class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

Many to many:

```php
class Post extends Model
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
}

$post->tags()->attach($tagId);
$post->tags()->detach($tagId);
$post->tags()->sync([1, 2, 3]);
```

Has one:

```php
public function profile()
{
    return $this->hasOne(Profile::class);
}
```

Polymorphic:

```php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

Eager loading:

```php
Post::with('user', 'tags')->get();
Post::withCount('comments')->get();
Post::with(['comments' => fn ($q) => $q->latest()])->get();
```

Avoid N+1:

```php
// Bad inside loop: $post->user
$posts = Post::with('user')->latest()->paginate(20);
```

## Migrations

Create migration:

```bash
php artisan make:migration create_posts_table
php artisan migrate
php artisan migrate:status
php artisan migrate:rollback
php artisan migrate:rollback --step=1
php artisan migrate:fresh
php artisan migrate:fresh --seed
```

Schema example:

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->string('slug')->unique();
    $table->text('body');
    $table->boolean('is_published')->default(false);
    $table->timestamp('published_at')->nullable();
    $table->json('meta')->nullable();
    $table->timestamps();
    $table->softDeletes();

    $table->index(['user_id', 'created_at']);
});
```

Column types:

```php
$table->id();
$table->uuid('uuid');
$table->string('name');
$table->text('body');
$table->integer('views');
$table->unsignedBigInteger('user_id');
$table->decimal('amount', 10, 2);
$table->boolean('active');
$table->date('birthday');
$table->dateTime('starts_at');
$table->timestamp('published_at')->nullable();
$table->json('settings');
$table->timestamps();
$table->softDeletes();
```

Foreign keys:

```php
$table->foreignId('user_id')->constrained();
$table->foreignId('user_id')->constrained('users')->cascadeOnDelete();
$table->foreignIdFor(User::class)->constrained()->restrictOnDelete();
```

## Seeders And Factories

Factory:

```php
class PostFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title' => fake()->sentence(),
            'body' => fake()->paragraphs(3, true),
            'is_published' => fake()->boolean(),
        ];
    }
}
```

Seeder:

```php
User::factory()
    ->has(Post::factory()->count(5))
    ->count(10)
    ->create();
```

Run seeders:

```bash
php artisan db:seed
php artisan db:seed --class=UserSeeder
php artisan migrate:fresh --seed
```

## Blade

Echo:

```blade
{{ $name }}
{!! $html !!}
```

Conditionals:

```blade
@if ($user)
    Hello {{ $user->name }}
@elseif ($guest)
    Guest
@else
    Unknown
@endif
```

Loops:

```blade
@foreach ($posts as $post)
    {{ $post->title }}
@endforeach

@forelse ($posts as $post)
    {{ $post->title }}
@empty
    No posts found.
@endforelse
```

Layouts:

```blade
@extends('layouts.app')

@section('content')
    Page content
@endsection
```

Components:

```blade
<x-alert type="success" :message="$message" />
```

CSRF and methods:

```blade
@csrf
@method('PUT')
```

## Authentication And Authorization

Useful auth helpers:

```php
auth()->user();
Auth::user();
$request->user();
auth()->id();
auth()->check();
auth()->logout();
```

Route protection:

```php
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', DashboardController::class);
});
```

Policy:

```bash
php artisan make:policy PostPolicy --model=Post
```

Policy method:

```php
public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}
```

Usage:

```php
$this->authorize('update', $post);

@can('update', $post)
    Edit
@endcan
```

## API Development

JSON response:

```php
return response()->json([
    'data' => $posts,
]);
```

API resource:

```bash
php artisan make:resource PostResource
```

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'created_at' => $this->created_at?->toISOString(),
        ];
    }
}
```

Return resource:

```php
return new PostResource($post);
return PostResource::collection($posts);
```

API validation errors return HTTP 422.

Common API status codes:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthenticated
403 Forbidden
404 Not Found
422 Validation Error
500 Server Error
```

## Middleware

Create:

```bash
php artisan make:middleware EnsureUserIsAdmin
```

Example:

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->user()?->is_admin) {
        abort(403);
    }

    return $next($request);
}
```

Use on route:

```php
Route::middleware(['auth', 'admin'])->group(function () {
    //
});
```

## Jobs And Queues

Set queue driver:

```env
QUEUE_CONNECTION=database
```

Create jobs table:

```bash
php artisan queue:table
php artisan migrate
```

Create job:

```bash
php artisan make:job ProcessPodcast
```

Dispatch:

```php
ProcessPodcast::dispatch($podcast);
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(10));
```

Run worker:

```bash
php artisan queue:work
php artisan queue:work --queue=high,default
php artisan queue:work --tries=3
php artisan queue:restart
```

Failed jobs:

```bash
php artisan queue:failed
php artisan queue:retry all
php artisan queue:flush
```

Production workers should be managed by Supervisor, systemd, Docker, or the hosting platform. Restart workers after deploy because workers keep old code in memory.

## Events And Listeners

Create:

```bash
php artisan make:event OrderPlaced
php artisan make:listener SendOrderPlacedNotification --event=OrderPlaced
```

Dispatch:

```php
event(new OrderPlaced($order));
OrderPlaced::dispatch($order);
```

Listener:

```php
public function handle(OrderPlaced $event): void
{
    // Send notification or update state.
}
```

## Mail And Notifications

Mail:

```bash
php artisan make:mail WelcomeMail
```

```php
Mail::to($user->email)->send(new WelcomeMail($user));
Mail::to($user)->queue(new WelcomeMail($user));
```

Notification:

```bash
php artisan make:notification InvoicePaid
```

```php
$user->notify(new InvoicePaid($invoice));
Notification::send($users, new InvoicePaid($invoice));
```

## Cache

```php
Cache::put('key', 'value', now()->addMinutes(10));
Cache::get('key');
Cache::remember('posts', 600, fn () => Post::latest()->get());
Cache::forget('key');
Cache::flush();
```

Cache tags are not supported by every cache driver.

## Files And Storage

Storage disk:

```php
Storage::disk('public')->put('avatars/user.jpg', $contents);
Storage::disk('public')->url('avatars/user.jpg');
Storage::disk('public')->delete('avatars/user.jpg');
```

Public storage link:

```bash
php artisan storage:link
```

Upload:

```php
$path = $request->file('avatar')->store('avatars', 'public');
```

## Logging

```php
Log::debug('Debug message', ['id' => $id]);
Log::info('User logged in', ['user_id' => $user->id]);
Log::warning('Payment retry needed');
Log::error('Payment failed', ['exception' => $e]);
```

Log file:

```text
storage/logs/laravel.log
```

## Testing

Run tests:

```bash
php artisan test
php artisan test --filter=PostTest
php artisan test --parallel
```

Feature test:

```php
public function test_user_can_view_posts(): void
{
    $user = User::factory()->create();
    Post::factory()->count(3)->create();

    $this->actingAs($user)
        ->get('/posts')
        ->assertOk()
        ->assertSee('Posts');
}
```

Database test:

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class PostTest extends TestCase
{
    use RefreshDatabase;
}
```

Useful assertions:

```php
assertOk()
assertCreated()
assertNoContent()
assertRedirect()
assertStatus(422)
assertJsonValidationErrors()
assertDatabaseHas()
assertDatabaseMissing()
```

## Common Laravel Problems

Route not found:

```bash
php artisan route:list
php artisan route:clear
```

Config changes not working:

```bash
php artisan config:clear
php artisan optimize:clear
```

Class not found:

```bash
composer dump-autoload
```

Storage files not public:

```bash
php artisan storage:link
```

Permission issue on Linux:

```bash
chmod -R ug+rw storage bootstrap/cache
```

Laravel cannot connect to MySQL:

```text
Check DB_HOST, DB_PORT, DB_DATABASE, DB_USERNAME, DB_PASSWORD.
Run php artisan config:clear after changing .env.
Use 127.0.0.1 instead of localhost if socket resolution causes issues.
```

## Deployment Checklist

```bash
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan queue:restart
```

Production `.env`:

```env
APP_ENV=production
APP_DEBUG=false
```

Do not commit:

```text
.env
vendor/
node_modules/
storage/logs/
```


---

# Source: php-composer-knowledge.md

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


---

# Source: frontend-vite-knowledge.md

# Frontend And Vite Knowledge

Use this file as Laravel frontend, Blade, Vite, JavaScript, CSS, and asset troubleshooting training/reference data.

## Laravel Vite Commands

Install packages:

```bash
npm install
```

Development:

```bash
npm run dev
```

Production build:

```bash
npm run build
```

Blade:

```blade
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

## Common Vite Problems

`Vite manifest not found`:

```text
Run npm run build for production.
Run npm run dev for local development.
Check public/build exists after build.
```

CSS not updating:

```bash
php artisan view:clear
npm run dev
```

JS error:

```text
Open browser devtools console.
Check network tab for failed assets.
Check terminal running npm run dev.
```

## JavaScript Basics

Variables:

```js
const name = 'Laravel';
let count = 0;
```

Fetch API:

```js
const response = await fetch('/api/posts');
const data = await response.json();
```

POST JSON:

```js
await fetch('/api/posts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
  },
  body: JSON.stringify({ title: 'Post title' }),
});
```

## CSS Layout Basics

Flex:

```css
.row {
  display: flex;
  gap: 1rem;
  align-items: center;
}
```

Grid:

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1rem;
}
```

Responsive:

```css
@media (max-width: 768px) {
  .sidebar {
    display: none;
  }
}
```

## Blade And Frontend Data

Pass data to Blade:

```php
return view('posts.index', [
    'posts' => Post::latest()->paginate(15),
]);
```

Render:

```blade
@foreach ($posts as $post)
    <article>
        <h2>{{ $post->title }}</h2>
        <p>{{ $post->excerpt }}</p>
    </article>
@endforeach

{{ $posts->links() }}
```

## Forms

```blade
<form method="POST" action="{{ route('posts.store') }}">
    @csrf

    <input name="title" value="{{ old('title') }}">

    @error('title')
        <p>{{ $message }}</p>
    @enderror

    <button type="submit">Save</button>
</form>
```

Update form:

```blade
<form method="POST" action="{{ route('posts.update', $post) }}">
    @csrf
    @method('PUT')
</form>
```

Delete form:

```blade
<form method="POST" action="{{ route('posts.destroy', $post) }}">
    @csrf
    @method('DELETE')
    <button type="submit">Delete</button>
</form>
```

## Accessibility Basics

- Use real buttons for actions.
- Use links for navigation.
- Every input should have a label.
- Images should have useful `alt` text or empty `alt=""` if decorative.
- Do not rely only on color to communicate state.
- Keep focus states visible.
- Make errors readable near the related input.

## Frontend Debug Checklist

- Browser console has no JavaScript errors.
- Network tab shows assets loading with HTTP 200.
- `npm run dev` is running for local Vite mode.
- `npm run build` was run for production.
- Blade has correct `@vite` paths.
- Clear view cache if Blade output looks stale.
- For NativePHP/emulator, confirm device can reach Vite/server host.

