# Knowledge Base

This workspace contains Markdown training/reference notes for Laravel, NativePHP, PHP, MySQL, Android emulator debugging, APIs, deployment, frontend tooling, Git, and testing.

## Files

- `laravel-cheatsheet.md`: Laravel commands, routing, Eloquent, Blade, auth, queues, APIs, cache, storage, tests, deployment.
- `mysql-optimization.md`: MySQL indexing, EXPLAIN, Laravel query optimization, transactions, pagination, database debugging.
- `nativephp-notes.md`: NativePHP mobile commands, Android setup, emulator networking, build/debug guidance.
- `emulator-fixes.md`: Android Emulator, ADB, GPU, networking, and NativePHP/Laravel mobile fixes.
- `debugging-notes.md`: General Laravel/PHP/Composer/MySQL/Vite/NativePHP troubleshooting.
- `php-composer-knowledge.md`: PHP syntax, Composer, autoloading, extensions, quality habits.
- `git-workflow-knowledge.md`: Git commands, branches, commits, conflicts, stash, ignore rules.
- `api-security-knowledge.md`: REST APIs, auth, authorization, validation, SQL injection, XSS, CSRF, CORS.
- `deployment-ops-knowledge.md`: Deployment, queues, scheduler, Linux/Windows ops, backups, monitoring.
- `frontend-vite-knowledge.md`: Vite, Blade frontend, JS, CSS, forms, accessibility, asset debugging.
- `testing-quality-knowledge.md`: Laravel tests, factories, assertions, review checklist, refactoring rules.

## GPT Usage Guidance

When using this as GPT training/reference context:

- Prefer Laravel conventions before custom patterns.
- Debug from logs and exact errors, not guesses.
- Validate and authorize user input.
- Use Eloquent relationships carefully and avoid N+1 queries.
- Use queues for slow or unreliable work.
- Use `10.0.2.2` for Android emulator access to host localhost.
- Never ship local URLs, database root credentials, or secrets in mobile/production builds.
- Keep production `APP_DEBUG=false`.
- Add tests for important behavior.
