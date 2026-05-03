# MySQL Optimization For Laravel

Use this file as MySQL, database, and Laravel query performance training/reference data.

## Core Rules

Always start with measurement:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@example.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@example.com';
```

Basic rules:

- Index columns used in `WHERE`, `JOIN`, `ORDER BY`, and frequent filters.
- Avoid `SELECT *`; select only columns needed.
- Avoid N+1 queries in Laravel by using eager loading.
- Use pagination for large lists.
- Use `LIMIT` for previews and admin search screens.
- Do not put functions around indexed columns in `WHERE` when avoidable.
- Prefer integer primary keys and foreign keys for joins.
- Keep transactions short.
- Use queue jobs for slow write-heavy or external-service work.
- Archive or partition very large historical tables if queries only need recent data.

## Index Basics

Single column index:

```sql
CREATE INDEX idx_users_email ON users(email);
```

Composite index:

```sql
CREATE INDEX idx_orders_user_status_created ON orders(user_id, status, created_at);
```

Unique index:

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

Laravel migration:

```php
Schema::table('users', function (Blueprint $table) {
    $table->index('email');
    $table->unique('email');
    $table->index(['user_id', 'status', 'created_at']);
});
```

Foreign key with index:

```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

## Composite Index Order

Index order matters. Good composite index order usually follows:

```text
Equality filters first, then range filters, then ordering.
```

Example query:

```sql
SELECT *
FROM orders
WHERE user_id = 10
  AND status = 'paid'
  AND created_at >= '2026-01-01'
ORDER BY created_at DESC;
```

Good index:

```sql
CREATE INDEX idx_orders_user_status_created ON orders(user_id, status, created_at);
```

Why:

```text
user_id is equality
status is equality
created_at is range/order
```

## Bad Query Patterns

Bad:

```sql
SELECT * FROM users;
```

Better:

```sql
SELECT id, name, email FROM users;
```

Bad because function can block index usage:

```sql
SELECT * FROM orders WHERE DATE(created_at) = '2026-05-01';
```

Better:

```sql
SELECT * FROM orders
WHERE created_at >= '2026-05-01 00:00:00'
  AND created_at < '2026-05-02 00:00:00';
```

Bad leading wildcard:

```sql
SELECT * FROM users WHERE name LIKE '%john%';
```

Better if prefix search is enough:

```sql
SELECT * FROM users WHERE name LIKE 'john%';
```

For real full-text search:

```sql
ALTER TABLE posts ADD FULLTEXT idx_posts_title_body (title, body);
SELECT * FROM posts WHERE MATCH(title, body) AGAINST('laravel mysql');
```

## Laravel Query Optimization

Avoid N+1:

```php
// Bad
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name;
}

// Good
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name;
}
```

Load counts efficiently:

```php
$posts = Post::withCount('comments')->get();
```

Constrain eager loading:

```php
$users = User::with(['posts' => function ($query) {
    $query->where('status', 'published')->latest();
}])->get();
```

Select fewer columns:

```php
$users = User::query()
    ->select(['id', 'name', 'email'])
    ->where('active', true)
    ->get();
```

Use `exists()` instead of `count()` when only checking existence:

```php
if (User::where('email', $email)->exists()) {
    //
}
```

Use `value()` when only one column is needed:

```php
$email = User::whereKey($id)->value('email');
```

Use chunking for large datasets:

```php
User::chunkById(1000, function ($users) {
    foreach ($users as $user) {
        // process
    }
});
```

Use cursors for memory-friendly iteration:

```php
foreach (User::where('active', true)->cursor() as $user) {
    // process one at a time
}
```

Use bulk insert/update where possible:

```php
User::insert($rows);

User::whereIn('id', $ids)->update(['active' => false]);
```

## Pagination

Normal pagination:

```php
Post::latest()->paginate(15);
```

Simple pagination avoids total count:

```php
Post::latest()->simplePaginate(15);
```

Cursor pagination works well for large tables:

```php
Post::orderBy('id')->cursorPaginate(15);
```

Avoid deep offset pagination on huge tables:

```sql
SELECT * FROM posts ORDER BY id LIMIT 15 OFFSET 500000;
```

Prefer keyset/cursor style:

```sql
SELECT * FROM posts WHERE id > 500000 ORDER BY id LIMIT 15;
```

## EXPLAIN Notes

Important EXPLAIN columns:

```text
type: access type; const/ref/range are usually better than ALL
key: index actually used
rows: estimated rows scanned
Extra: filesort, temporary, using index, using where
```

Warning signs:

```text
type = ALL
key = NULL
rows is very high
Extra has Using temporary
Extra has Using filesort on large result
```

## Joins

Index foreign keys:

```sql
CREATE INDEX idx_posts_user_id ON posts(user_id);
```

Join example:

```sql
SELECT posts.id, posts.title, users.name
FROM posts
JOIN users ON users.id = posts.user_id
WHERE users.active = 1;
```

Indexes:

```sql
users(id)
users(active)
posts(user_id)
```

In Laravel:

```php
Post::query()
    ->join('users', 'users.id', '=', 'posts.user_id')
    ->where('users.active', true)
    ->select('posts.id', 'posts.title', 'users.name')
    ->get();
```

Use Eloquent relationships for readability, but use query builder joins for reports when you need exact SQL and fewer hydrated models.

## Aggregates And Reports

```php
Order::where('status', 'paid')->sum('total');
Order::whereDate('created_at', today())->count();
Order::selectRaw('status, COUNT(*) as total')->groupBy('status')->get();
```

Avoid `whereDate` on huge indexed timestamp columns when performance matters:

```php
Order::where('created_at', '>=', $start)
    ->where('created_at', '<', $end)
    ->count();
```

For dashboards, cache expensive aggregates:

```php
$stats = Cache::remember('dashboard.stats', 300, function () {
    return [
        'users' => User::count(),
        'orders' => Order::where('status', 'paid')->count(),
    ];
});
```

## Transactions And Locks

Transaction:

```php
DB::transaction(function () use ($order) {
    $order->update(['status' => 'paid']);
    Payment::create([...]);
});
```

Pessimistic lock:

```php
DB::transaction(function () use ($id) {
    $wallet = Wallet::whereKey($id)->lockForUpdate()->first();
    $wallet->decrement('balance', 100);
});
```

Tips:

- Keep transactions short.
- Do not call slow external APIs inside a transaction.
- Update rows in a consistent order to reduce deadlocks.
- Retry deadlocked transactions when appropriate.

## Data Types

Good choices:

```text
Use unsigned bigint for Laravel default IDs.
Use decimal for money, not float.
Use boolean for flags.
Use timestamp/datetime for dates.
Use json only when you do not need frequent indexed filtering.
Use varchar length that matches real data where practical.
```

Money:

```php
$table->decimal('amount', 10, 2);
```

Boolean:

```php
$table->boolean('active')->default(true);
```

JSON:

```php
$table->json('settings')->nullable();
```

## MySQL Configuration Concepts

Important server concepts:

```text
innodb_buffer_pool_size: memory for InnoDB data/index cache
max_connections: max concurrent client connections
slow_query_log: logs slow queries
long_query_time: threshold for slow query log
tmp_table_size and max_heap_table_size: memory temp table limits
```

For production, enable slow query logging and inspect repeated slow queries before adding indexes blindly.

## Laravel Database Debugging

Show SQL for a query:

```php
$query = User::where('email', $email);
dd($query->toSql(), $query->getBindings());
```

Listen to queries locally:

```php
DB::listen(function ($query) {
    logger()->debug($query->sql, $query->bindings);
});
```

Check database connection:

```bash
php artisan tinker
DB::connection()->getPdo();
```

Common fixes:

```bash
php artisan config:clear
php artisan optimize:clear
```

## Practical Index Examples

Users login by email:

```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

Posts by author:

```sql
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at);
```

Orders by customer/status:

```sql
CREATE INDEX idx_orders_user_status_created ON orders(user_id, status, created_at);
```

Soft deletes with common active query:

```sql
CREATE INDEX idx_posts_deleted_created ON posts(deleted_at, created_at);
```

Search by slug:

```sql
CREATE UNIQUE INDEX idx_posts_slug ON posts(slug);
```

## Checklist Before Adding An Index

- Is the query slow in real usage?
- Is the query frequent?
- Does `EXPLAIN` show a table scan or poor index?
- Is the column selective enough?
- Will a composite index cover multiple common filters?
- Will the new index slow down heavy writes too much?
- Is there already a similar index?

## Common Laravel Performance Checklist

- Use eager loading for relationships.
- Use `withCount` instead of loading all children just to count.
- Use `select` to avoid loading large unused columns.
- Use `simplePaginate` or `cursorPaginate` for large tables.
- Use indexes for common filters and joins.
- Cache expensive dashboard queries.
- Move slow jobs to queues.
- Avoid querying inside Blade loops.
- Avoid calling external APIs during HTTP request if queue is possible.
- Use production caches: config, route, view.
