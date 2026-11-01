---
name: ci-gate
description: 'Add a single gate job to GitHub Actions CI that needs every other check job, so the repository ruleset requires only that one status check and jobs can be added or renamed in code. Use when CI has several jobs, when required status checks list many jobs, when a renamed job blocks merges, or when adding jobs to CI.'
argument-hint: 'Optional workflow file to update'
---

# CI gate job

When a workflow has more than one job that should block merging, a single `<file>-required` job depends on all of them, where `<file>` is the workflow file name without `.yml` (`ci-required` in `ci.yml`). The ruleset requires only the gate, so adding, removing, or renaming jobs is a code change, not a settings change.

## Procedure

1. `needs` only works within a workflow, so each workflow with blocking jobs gets its own gate. Prefer keeping check jobs that run on `pull_request` together in `ci.yml` so there's one gate to require.
2. Add the gate as the last job (here in `ci.yml`):

   ```yaml
   ci-required:
     if: ${{ !cancelled() }}
     needs: [lint, test, build]
     runs-on: ubuntu-latest
     permissions: {}
     steps:
       - if: ${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') }}
         run: exit 1
   ```

   - `needs` lists every job in the workflow that runs on pull requests and should block merging. Leave out deploy jobs that only run on the default branch.
   - The `if:` keeps the gate running when a dependency fails. Without it the gate is skipped, and rulesets treat a skipped required check as passing.
   - Skipped dependencies (path filters, conditional jobs) pass. If a job must never be skipped, also check `needs.<job>.result == 'skipped'` for it.

3. Make sure the workflow triggers on every event the ruleset checks: `pull_request`, plus `merge_group` if the repo uses a merge queue.
4. When adding a job to the workflow later, add it to the gate's `needs` in the same change. If the workflow file is renamed, rename the gate to match and tell the user the required check name changed.
5. Rerun `gh actions-lock` if the repo has `.github/workflows/actions.lock`.

## Ruleset

Don't change rulesets yourself. Read the current required checks with `gh api repos/{owner}/{repo}/rulesets` (and `gh api repos/{owner}/{repo}/rulesets/<id>` for details), then tell the user to:

1. Merge the workflow change first so the `<file>-required` check exists.
2. Set the ruleset's required status checks to just the gate jobs (source: GitHub Actions), removing the individual job checks.

## Verify

- Every job that runs on `pull_request`, except the gate, appears in its workflow's `<file>-required` `needs`.
- `actionlint` passes on the workflow if it's installed.
