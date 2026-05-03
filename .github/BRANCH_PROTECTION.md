# Branch Protection Setup

Use these GitHub settings to protect the `master` branch.

## GitHub UI Path

```text
Repository -> Settings -> Branches -> Add branch protection rule
```

Branch name pattern:

```text
master
```

## Recommended Settings

Enable:

- Require a pull request before merging.
- Require approvals.
- Required number of approvals before merging: `1`.
- Dismiss stale pull request approvals when new commits are pushed.
- Require review from Code Owners.
- Require conversation resolution before merging.
- Do not allow bypassing the above settings.
- Restrict who can push to matching branches.

Optional if you add CI later:

- Require status checks to pass before merging.
- Require branches to be up to date before merging.

This repo includes a GitHub Actions workflow:

```text
.github/workflows/markdown-check.yml
```

Required status check name:

```text
markdown-check
```

## Code Owner

The repo uses `.github/CODEOWNERS`.

Current owner:

```text
@souravlai7
```

This means pull requests require review from `souravlai7` when GitHub branch protection has "Require review from Code Owners" enabled.

## Recommended Workflow

```text
1. Contributor forks the repo.
2. Contributor creates a feature branch.
3. Contributor edits source Markdown files.
4. Contributor regenerates upload-ready files if needed.
5. Contributor opens PR into master.
6. souravlai7 reviews the PR.
7. PR is merged only after approval and resolved conversations.
```

## Avoid Direct Pushes

Do not push directly to `master`.

Use feature branches:

```bash
git switch -c update-nativephp-sync-notes
git add .
git commit -m "Improve NativePHP offline sync notes"
git push -u origin update-nativephp-sync-notes
```

Then open a pull request into `master`.
