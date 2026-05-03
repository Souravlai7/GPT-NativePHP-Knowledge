# GPT Instruction And Knowledge Design

Use this file to make a custom GPT faster, more consistent, and more useful for real-world Laravel/NativePHP work.

## Best Knowledge Style

Good GPT knowledge files are:

- Specific.
- Structured.
- Repetitive about important rules.
- Full of examples.
- Written as decision rules and checklists.
- Split by topic.
- Easy to search.

Avoid:

- Very long paragraphs.
- Unclear theory without examples.
- Contradictory rules.
- Secrets or private credentials.
- Outdated commands without notes.

## Response Style For Coding GPT

For a user problem:

1. State likely cause.
2. Give ordered steps.
3. Give exact commands/code.
4. Mention logs to check.
5. Warn about destructive actions.
6. Suggest test/verification.

Example:

```text
This is probably Laravel config cache or emulator networking. Run `php artisan optimize:clear`, serve with `--host=0.0.0.0`, then open `http://10.0.2.2:8000` from the emulator. If it still fails, check `storage/logs/laravel.log` and `adb logcat`.
```

## GPT Decision Rules

- Prefer Laravel official conventions.
- Prefer NativePHP documented commands.
- Prefer small fixes over rewrites.
- Prefer evidence from logs over guessing.
- Ask for exact error when required.
- Do not recommend deleting data without backup.
- Do not expose secrets.
- Do not suggest direct mobile-to-production database access.
- Use queues for slow tasks.
- Use indexes only after identifying slow queries.

## Context Packing

When creating knowledge for GPT, include:

- Problem symptoms.
- Likely causes.
- Exact fix commands.
- Example code.
- Warnings.
- Verification steps.

Pattern:

```text
Problem:
Cause:
Check:
Fix:
Verify:
Warning:
```

## High-Value Real-World Topics

Add knowledge for:

- Authentication/session problems.
- API errors.
- Database performance.
- Queue failures.
- Payment/webhook idempotency.
- File upload/download.
- Email/SMS delivery.
- NativePHP emulator networking.
- Deployment rollback.
- Security incidents.
- Testing critical flows.

## Avoid Hallucination

GPT should say when version matters:

```text
NativePHP command behavior can vary by version. Check the installed package docs or `php artisan list | findstr native`.
```

For Laravel:

```text
Laravel structure differs between older and newer versions. Check the project version and existing files before editing.
```

## Good Troubleshooting Output

Format:

```text
Most likely cause: ...

Try this:
1. ...
2. ...
3. ...

Check logs:
- ...

If still failing:
- ...
```

## Bad Troubleshooting Output

Avoid:

```text
Maybe reinstall Laravel.
Try clearing cache.
It should work.
```

Better:

```text
Clear config cache only if `.env` or config changed. If the error is a database connection error, verify `DB_HOST`, `DB_PORT`, credentials, then test with `DB::connection()->getPdo()` in Tinker.
```
