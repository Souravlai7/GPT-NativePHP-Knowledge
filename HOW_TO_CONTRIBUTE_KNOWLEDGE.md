# How To Contribute GPT Knowledge

This guide explains exactly how users should contribute knowledge to this repo.

## Quick Summary

```text
Open issue -> fork repo -> create branch -> edit source Markdown -> regenerate upload-ready -> commit -> PR -> link issue
```

## Step 1: Open An Issue

Before writing a pull request, open an issue first.

Use an issue for:

- New knowledge topic.
- Bug in existing knowledge.
- Outdated Laravel or NativePHP command.
- Unsafe advice that should be fixed.
- Missing GPT instruction or conversation starter.

Good issue title examples:

```text
[Knowledge]: Add NativePHP camera permission troubleshooting
[Bug]: Fix outdated Android emulator GPU flag note
[Knowledge]: Add Laravel Reverb private channel auth checklist
```

## Step 2: Fork The Repo

Click `Fork` on GitHub, then clone your fork.

```bash
git clone https://github.com/YOUR-USERNAME/GPT-NativePHP-Knowledge.git
cd GPT-NativePHP-Knowledge
```

## Step 3: Create A Branch

Use a short descriptive branch name.

```bash
git switch -c add-nativephp-camera-notes
```

Good branch names:

```text
add-nativephp-camera-notes
fix-reverb-emulator-host
improve-payment-webhook-guide
add-sqlite-sync-conflicts
```

## Step 4: Edit Source Markdown Files

Edit source files in the repo root, for example:

```text
nativephp-notes.md
nativephp-advanced-knowledge.md
nativephp-offline-api-payment.md
socket-realtime-knowledge.md
mysql-optimization.md
debugging-notes.md
```

Do not edit only `upload-ready/`. Those are merged files for GPT upload.

## Step 5: Write Practical Knowledge

Good contribution format:

```text
Problem:
Likely causes:
Check:
Fix:
Verify:
Warning:
```

Include practical examples:

```bash
php artisan optimize:clear
php artisan route:list
adb logcat
```

## Step 6: Regenerate Upload-Ready Files

If source Markdown files changed, regenerate the merged files in `upload-ready/` before opening a PR.

If you cannot regenerate them, mention that in the PR.

## Step 7: Commit Changes

```bash
git status
git add .
git commit -m "Add NativePHP camera permission notes"
```

Good commit messages:

```text
Add NativePHP offline sync conflict guide
Fix Laravel Reverb emulator socket notes
Improve payment webhook idempotency examples
Add MySQL composite index examples
```

Avoid:

```text
fix
update
changes
final
```

## Step 8: Push And Open Pull Request

```bash
git push -u origin add-nativephp-camera-notes
```

Open a pull request into:

```text
souravlai7/GPT-NativePHP-Knowledge -> master
```

## Step 9: Link The Issue

In the PR description, include one of these:

```text
Closes #12
Fixes #18
Refs #21
```

Use `Closes` or `Fixes` when the PR fully resolves the issue. Use `Refs` when it partially relates.

## What We Accept

- Real Laravel/NativePHP fixes.
- Debugging checklists.
- Safe production guidance.
- Commands with verification steps.
- Offline sync/API/payment/socket examples.
- Version-specific notes.
- Better GPT instructions and conversation starters.

## What We Do Not Accept

- Secrets or credentials.
- Private customer/project data.
- Unsafe payment shortcuts.
- Direct production database credentials in mobile apps.
- Destructive commands without warnings.
- Vague advice without checks or examples.

## Maintainer

Current maintainer:

```text
@souravlai7
```
