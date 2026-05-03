# Contributing

Thanks for helping improve this Laravel and NativePHP GPT knowledge base.

## What To Contribute

Useful contributions include:

- Laravel troubleshooting notes.
- NativePHP setup, build, emulator, and release fixes.
- Offline sync patterns.
- Local SQLite/database examples.
- API calling patterns.
- Payment and webhook safety patterns.
- Socket/Reverb/Echo troubleshooting.
- MySQL optimization examples.
- Deployment and production incident runbooks.
- Clear GPT instructions, descriptions, and conversation starters.

## Required Workflow

Every contribution should start with an issue.

```text
1. Open or claim an issue.
2. Wait for maintainer feedback if the change is large or risky.
3. Fork the repo.
4. Create a feature branch.
5. Edit source Markdown files in the repo root.
6. Regenerate upload-ready files if source files changed.
7. Commit and push your branch.
8. Open a pull request into master.
9. Link the issue in the PR description.
```

Use one of these in the PR description:

```text
Closes #12
Fixes #18
Refs #21
```

## Contribution Rules

- Create or claim an issue before opening a pull request.
- Every pull request should link to an issue using `Closes #issue-number`, `Fixes #issue-number`, or `Refs #issue-number`.
- Edit the source Markdown files in the repo root first.
- Keep `upload-ready/` files as merged upload files.
- Do not add secrets, API keys, passwords, private URLs, customer data, or screenshots with private data.
- Prefer practical examples over theory.
- Include likely cause, check, fix, and verification steps where possible.
- Prefer Laravel and NativePHP official conventions.
- Mark version-specific behavior clearly.
- Add warnings before destructive commands.
- Keep examples generic and reusable.

## Writing Style

Use this pattern for troubleshooting knowledge:

```text
Problem:
Likely causes:
Check:
Fix:
Verify:
Warning:
```

Use fenced code blocks for commands and code:

````md
```bash
php artisan optimize:clear
```
````

## Pull Request Checklist

- [ ] I created or claimed an issue before opening this PR.
- [ ] I linked the issue in the PR description.
- [ ] I edited the correct source Markdown file.
- [ ] I did not include secrets or private data.
- [ ] I added practical examples, commands, or checklists.
- [ ] I updated `README.md` if I added a new file or major topic.
- [ ] I regenerated `upload-ready/` if source files changed.
- [ ] I marked version-specific behavior where needed.

## Regenerating Upload Files

This repo keeps detailed source notes in the root and merged GPT upload files in `upload-ready/`.

If you change source files, regenerate the merged upload files before opening a pull request.

## Safety Rules

Do not contribute guidance that:

- Stores payment secrets in mobile apps.
- Recommends direct production database access from NativePHP/mobile apps.
- Disables security permanently.
- Deletes production data without backup.
- Retries payments without idempotency or provider verification.
- Logs passwords, tokens, private keys, card data, or API secrets.
