# Knowledge Base

This workspace contains Markdown training/reference notes for Laravel, NativePHP, PHP, MySQL, Android emulator debugging, APIs, deployment, frontend tooling, Git, testing, architecture, real-world troubleshooting, incidents, integrations, performance, and GPT instruction design.

## Files

- `gpt-description.md`: Polished custom GPT description, behavior instructions, scope, and conversation starters.
- `gpt-custom-instructions.md`: Paste-ready custom GPT Instructions for Laravel, NativePHP, offline sync, APIs, sockets, payments, debugging, and deployment.
- `gpt-conversation-starters.md`: Ready-to-use custom GPT conversation starters grouped by Laravel, NativePHP, offline sync, APIs, payments, sockets, performance, deployment, debugging, and security.
- `laravel-cheatsheet.md`: Laravel commands, routing, Eloquent, Blade, auth, queues, APIs, cache, storage, tests, deployment.
- `mysql-optimization.md`: MySQL indexing, EXPLAIN, Laravel query optimization, transactions, pagination, database debugging.
- `nativephp-notes.md`: NativePHP mobile commands, Android setup, emulator networking, build/debug guidance.
- `nativephp-auto-training.md`: Automatic response playbook for NativePHP posts/forms, API calls, app loading, database, emulator, and builds.
- `nativephp-advanced-knowledge.md`: Advanced NativePHP architecture, auth, database, assets, permissions, plugins, offline sync, builds, signing, and release guidance.
- `nativephp-offline-api-payment.md`: NativePHP local DB, offline-first sync, API clients, internet checks, local storage safety, payments, webhooks, and idempotency.
- `socket-realtime-knowledge.md`: Laravel Reverb, Echo, WebSockets, Socket.IO concepts, private/presence channels, NativePHP socket networking, and troubleshooting.
- `emulator-fixes.md`: Android Emulator, ADB, GPU, networking, and NativePHP/Laravel mobile fixes.
- `debugging-notes.md`: General Laravel/PHP/Composer/MySQL/Vite/NativePHP troubleshooting.
- `php-composer-knowledge.md`: PHP syntax, Composer, autoloading, extensions, quality habits.
- `git-workflow-knowledge.md`: Git commands, branches, commits, conflicts, stash, ignore rules.
- `api-security-knowledge.md`: REST APIs, auth, authorization, validation, SQL injection, XSS, CSRF, CORS.
- `deployment-ops-knowledge.md`: Deployment, queues, scheduler, Linux/Windows ops, backups, monitoring.
- `frontend-vite-knowledge.md`: Vite, Blade frontend, JS, CSS, forms, accessibility, asset debugging.
- `testing-quality-knowledge.md`: Laravel tests, factories, assertions, review checklist, refactoring rules.
- `realworld-problem-solving.md`: Practical diagnosis patterns, symptom-to-cause mapping, evidence ranking, rollback rules.
- `architecture-patterns.md`: Laravel architecture decisions, service/action classes, DTOs, repositories, anti-patterns.
- `incident-runbooks.md`: Production outage, database, queue, payment, email, file storage, and security incident runbooks.
- `performance-scalability.md`: Laravel, MySQL, API, queue, frontend, and NativePHP performance/scaling guidance.
- `integration-patterns.md`: External APIs, webhooks, payments, imports, exports, email, SMS/OTP, file storage.
- `gpt-instruction-knowledge.md`: How the custom GPT should reason, answer, avoid hallucination, and use knowledge.

## GPT Usage Guidance

When using this as GPT training/reference context:

- Prefer Laravel conventions before custom patterns.
- Debug from logs and exact errors, not guesses.
- Validate and authorize user input.
- Use Eloquent relationships carefully and avoid N+1 queries.
- Use queues for slow or unreliable work.
- Use `10.0.2.2` for Android emulator access to host localhost.
- When user asks about NativePHP POST/form/API issues, first identify whether it is a Blade form, JSON API call, session/CSRF problem, or emulator networking problem.
- For socket issues, separate connection failure from event delivery failure; then check host/port, auth, queues, channel names, and mobile/emulator networking.
- Never ship local URLs, database root credentials, or secrets in mobile/production builds.
- Keep production `APP_DEBUG=false`.
- Add tests for important behavior.
- Use exact logs, commands, and reproducible facts before guessing.
- For production incidents, mitigate user impact first, then root-cause.
- Keep mobile/native production apps away from local URLs and direct database credentials.
- For offline NativePHP apps, use local UUIDs, remote IDs, sync status fields, idempotent APIs, retry/backoff, and conflict handling.
- For payments, keep secret keys and final verification on the backend; use webhooks and idempotency keys.
- Prefer small, reversible fixes for real-world bugs.
- For custom GPT answers, use ordered checklists with concrete commands and verification steps.

## Upload Ready Files

The `upload-ready/` folder contains merged Markdown files for custom GPT upload limits.

Upload the numbered files from `upload-ready/` when a GPT builder limits the number of files. Keep the root Markdown files as editable source notes.

## Contributing

Contributions are welcome.

Before contributing:

- Read `CONTRIBUTING.md`.
- Edit source Markdown files in the repo root.
- Do not add secrets, passwords, API keys, private URLs, customer data, or payment secrets.
- Prefer practical examples with commands, code, checks, fixes, and verification steps.
- Update `README.md` when adding a new file or major topic.
- Regenerate `upload-ready/` when source knowledge changes.

## License

This knowledge base is licensed under `CC BY 4.0`. You can share and adapt it, including for custom GPTs, as long as you provide attribution.
