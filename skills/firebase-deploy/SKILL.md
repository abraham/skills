---
name: firebase-deploy
description: 'Set up or fix GitHub Actions deploys for a Firebase project using abraham/firebase-wif (Workload Identity Federation) instead of FIREBASE_TOKEN or service account key secrets. Use when a repo has firebase.json or .firebaserc and needs a deploy workflow, preview channels on pull requests, or a migration off token/key secrets.'
argument-hint: 'Optional workflow file to update'
---

# Firebase deploys with abraham/firebase-wif

Every Firebase deploy from GitHub Actions authenticates with `abraham/firebase-wif`. No `FIREBASE_TOKEN`, `GCP_SA_KEY`, or `FIREBASE_SERVICE_ACCOUNT*` secrets.

## Procedure

1. Follow the upstream `firebase-wif` skill for the workflow edits (permissions, fork guard, environments, auth step, deploy step, clean-up). If it isn't installed, read it from https://raw.githubusercontent.com/abraham/firebase-wif/main/skills/firebase-wif/SKILL.md, or install it with `gh skill install abraham/firebase-wif`.
2. Pin the latest release: `npm view @abraham/firebase-wif version`. Use the full tag (`abraham/firebase-wif@v0.2.1`), never `@main` or a moving major.
3. Node: set up Node with `node-version-file` (see the `node-version` skill). Functions deploys build with `functions/package.json`.
4. Environment URL: every deploy job has an `environment`, so give it a `url` (see the `environment-url` skill).
   - Live: `https://<project-id>.web.app`, or the custom domain if the site has one.
   - Preview channels: the URL from the deploy step's output.
5. Concurrency: one deploy per target at a time (see the `deploy-concurrency` skill).
6. If no deploy workflow exists, create `.github/workflows/deploy.yml`: deploy live on push to the default branch, and to a preview channel on same-repo pull requests when the project has Hosting.
7. Rerun `gh actions-lock` after changing `uses:` lines (see the `actions-lock` skill).

## Example live job

```yaml
deploy:
  runs-on: ubuntu-latest
  environment:
    name: production
    url: https://my-project.web.app
  concurrency:
    group: deploy-production
    cancel-in-progress: false
  permissions:
    contents: read
    id-token: write
  steps:
    - uses: actions/checkout@v7
      with:
        persist-credentials: false
    - uses: actions/setup-node@v7
      with:
        node-version-file: package.json
    - run: npm ci
    - run: npm run build
    - uses: abraham/firebase-wif@v0.2.1
      with:
        workload_identity_provider: ${{ vars.FIREBASE_WIF_PROVIDER }}
        service_account: ${{ vars.FIREBASE_WIF_SERVICE_ACCOUNT }}
    - run: npx firebase-tools deploy --non-interactive
```

## Example preview job

```yaml
preview:
  if: github.event.pull_request.head.repo.full_name == github.repository
  runs-on: ubuntu-latest
  environment:
    name: preview
    url: ${{ steps.deploy.outputs.url }}
  concurrency:
    group: deploy-preview-${{ github.event.pull_request.number }}
    cancel-in-progress: true
  permissions:
    contents: read
    id-token: write
  steps:
    # checkout, setup-node, build, and abraham/firebase-wif as above
    - id: deploy
      run: |
        npx firebase-tools hosting:channel:deploy "pr-${{ github.event.number }}" --expires 7d --json > deploy.json
        echo "url=$(jq -r '.result[].url' deploy.json | head -n 1)" >> "$GITHUB_OUTPUT"
```

## Don't

- Don't run `npx @abraham/firebase-wif setup` without asking; it changes Google Cloud IAM and GitHub settings. Run `npx @abraham/firebase-wif status` (read-only) and tell the user if setup is needed.
- Don't delete repository secrets. List the ones no longer used so the user can remove them after a successful deploy.
