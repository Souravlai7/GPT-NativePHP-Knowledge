# Git Workflow Knowledge

Use this file as Git and collaboration training/reference data.

## Basics

Check status:

```bash
git status
```

See changes:

```bash
git diff
git diff --staged
```

Stage and commit:

```bash
git add file.php
git commit -m "Fix user import validation"
```

History:

```bash
git log --oneline --decorate --graph
git show HEAD
```

Branches:

```bash
git branch
git switch -c feature/nativephp-login
git switch main
```

Remote:

```bash
git fetch
git pull
git push
git push -u origin feature/nativephp-login
```

## Safe Daily Workflow

```bash
git status
git fetch
git switch main
git pull
git switch -c feature/task-name
```

Before committing:

```bash
git status
git diff
php artisan test
```

## Merge Conflicts

Conflict markers:

```text
<<<<<<< HEAD
current branch changes
=======
incoming changes
>>>>>>> branch-name
```

Resolve by editing the file to the desired final content, then:

```bash
git add conflicted-file.php
git commit
```

## Useful Commands

Undo unstaged file change:

```bash
git restore file.php
```

Unstage:

```bash
git restore --staged file.php
```

Show who changed a line:

```bash
git blame file.php
```

Search history:

```bash
git log --grep="payment"
git log -S "oldFunctionName"
```

Stash:

```bash
git stash
git stash pop
git stash list
```

## Commit Message Style

Good messages:

```text
Fix checkout validation for zero amount orders
Add NativePHP Android emulator troubleshooting notes
Improve user import duplicate email handling
```

Avoid vague messages:

```text
fix
changes
update
final
```

## Git Ignore

Usually ignored:

```text
/vendor
/node_modules
/.env
/storage/logs/*.log
/public/build
```

Usually committed:

```text
composer.json
composer.lock
package.json
package-lock.json
database/migrations
config files without secrets
```
