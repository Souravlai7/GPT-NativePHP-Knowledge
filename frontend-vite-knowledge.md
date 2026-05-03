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
