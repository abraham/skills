---
name: deploy-concurrency
description: 'Add a concurrency group to GitHub Actions deploy jobs so only one deploy to an environment runs at a time. Use when a workflow deploys (Firebase, Pages, npm publish, cloud), when deploys can overlap or finish out of order, or when adding a deploy job.'
argument-hint: 'Optional workflow file to update'
---

# One deploy at a time

Every job that deploys has a `concurrency` group, so two deploys to the same target never run at once and an older commit can't finish after a newer one.

## Procedure

1. Find deploy jobs in `.github/workflows/*.yml`: jobs with `environment`, or steps that deploy (`firebase deploy`, `hosting:channel:deploy`, `actions/deploy-pages`, `npm publish`, cloud CLIs).
2. Add `concurrency` to the deploy job, not the workflow, so lint and test jobs in the same workflow aren't held up:

   ```yaml
   deploy:
     environment:
       name: production
       url: https://example.com
     concurrency:
       group: deploy-production
       cancel-in-progress: false
   ```

   - Name the group after the target (`deploy-<environment>`), not the workflow, so every workflow that deploys there (push deploys, manual `workflow_dispatch` redeploys, releases) shares one queue.
   - Live and production deploys use `cancel-in-progress: false`; stopping a deploy halfway can leave the target half-updated. A newer pending run still replaces an older pending one, so the latest commit ends up deployed.
   - Use workflow-level `concurrency` only when every job in the workflow is part of the deploy.

3. Per-pull-request previews get one group per PR, and can cancel older runs because a newer commit supersedes them:

   ```yaml
   preview:
     concurrency:
       group: deploy-preview-${{ github.event.pull_request.number }}
       cancel-in-progress: true
   ```

4. Rerun `gh actions-lock` if the repo has `.github/workflows/actions.lock` and other `uses:` lines changed.

## Verify

- Every deploy job has a `concurrency.group`, and jobs deploying to the same target use the same group.
- `actionlint` passes on the changed files if it's installed.
