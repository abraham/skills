---
name: actions-lock
description: 'Pin GitHub Actions dependencies with the gh-actions-lock CLI (github/gh-actions-lock) and verify .github/workflows/actions.lock in the CI lint job. Use when adding a workflow lockfile, after changing any uses: line in .github/workflows, or when the lint job is missing lockfile verification.'
---

# GitHub Actions lockfile

Workflows are locked with `gh actions-lock`, and the `lint` job fails when `.github/workflows/actions.lock` is out of sync.

For CLI details, modes, and troubleshooting, use the upstream `gh-actions-lock` skill (`gh skill install github/gh-actions-lock`), or `gh actions-lock --help`.

## Procedure

1. Make sure the CLI is installed: `gh extension list | grep actions-lock`, else `gh extension install github/gh-actions-lock`.
2. Add verification to the `lint` job. Find the job named `lint` in the CI workflow; if none exists, add one to the workflow that runs on `pull_request` and pushes to the default branch.

   ```yaml
   lint:
     runs-on: ubuntu-latest
     permissions:
       contents: read
     env:
       GH_TOKEN: ${{ github.token }}
     steps:
       - uses: actions/checkout@v7
         with:
           persist-credentials: false
       - run: gh extension install github/gh-actions-lock --pin v0.4.0
       - run: gh actions-lock --verify
       # other lint steps (format:check, etc.)
   ```

   Pin the extension to the latest release: `gh release view --repo github/gh-actions-lock --json tagName -q .tagName`.
   If a standalone `verify-lockfile.yml` workflow exists, move its check into the `lint` job and delete that workflow.

3. Generate or update the lockfile from the repo root: `gh actions-lock`. Do this last, after every other workflow edit, so the lint job's own `uses:` lines are locked too.
4. `.github/workflows/actions.lock` belongs in the same commit as the workflow changes.

## Notes

- Rerun `gh actions-lock` whenever a workflow is added or a `uses:` changes; `--verify` fails otherwise. Use `--relock` only to advance moving refs on purpose.
- Local `./` actions that use remote actions block locking: pass `--migrate-local-actions` or `--ignore-local-actions` (ask which).
- Never use `--accept-moved` in automation, and don't add it without asking.
- Reusable workflows from other repos and container images aren't locked; mention them to the user if present.

## Verify

`gh actions-lock --verify` passes locally, and `git status` shows `actions.lock` changed alongside the workflows.
