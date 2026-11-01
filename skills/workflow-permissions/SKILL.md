---
name: workflow-permissions
description: 'Give every GitHub Actions workflow an explicit least-privilege GITHUB_TOKEN permissions block. Use when a workflow has no permissions key, uses write-all or broad defaults, when adding a job that needs id-token, pull-requests, pages, packages, or other write access, or when hardening workflows.'
argument-hint: 'Optional workflow file to update'
---

# Workflow permissions

Every workflow in `.github/workflows/` declares `permissions`, so the `GITHUB_TOKEN` never falls back to the repository or organization default.

## Procedure

1. Set read-only permissions at the top of each workflow, after `on:`:

   ```yaml
   permissions:
     contents: read
   ```

   Use `permissions: {}` when no job checks out code or reads the repo.

2. Jobs that need more get their own job-level `permissions`. Job-level permissions replace the workflow's, so list everything the job uses, including `contents: read`:

   ```yaml
   deploy:
     permissions:
       contents: read
       id-token: write
   ```

3. Grant only what the steps use:

   | Need                                                           | Permission                               |
   | -------------------------------------------------------------- | ---------------------------------------- |
   | OIDC (`abraham/firebase-wif`, cloud auth, npm trusted publish) | `id-token: write`                        |
   | PR comments, labels, reviews                                   | `pull-requests: write`                   |
   | Issue comments, labels                                         | `issues: write`                          |
   | Push commits or tags, create releases                          | `contents: write`                        |
   | GitHub Pages deploy                                            | `pages: write`, `id-token: write`        |
   | GitHub Packages / GHCR push                                    | `packages: write`                        |
   | Check runs                                                     | `checks: write`                          |
   | Commit statuses                                                | `statuses: write`                        |
   | Attestations                                                   | `attestations: write`, `id-token: write` |
   | Reading Actions runs or artifacts from other runs              | `actions: read`                          |

   Read the docs of each action in the job to confirm what it needs; don't add a permission "just in case".

4. Replace `permissions: write-all` and `read-all` with the explicit list.
5. Keep write permissions on the jobs that need them, not the workflow, so lint and test jobs stay read-only.

## Don't

- Don't give `pull_request_target` or `workflow_run` workflows write permissions on jobs that check out or run pull request code.
- Don't change repository or organization default token settings; that's for the user to do.

## Verify

- Every file in `.github/workflows/` has a top-level `permissions` key.
- Run `actionlint` and `zizmor` on the changed files if they're installed.
