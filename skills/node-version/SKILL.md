---
name: node-version
description: 'Set the Node.js version in package.json engines and make GitHub Actions read it with actions/setup-node node-version-file. Use when adding or upgrading Node, fixing hard-coded node-version in workflows, removing .nvmrc/.node-version duplication, or setting the Node runtime for Firebase Cloud Functions.'
argument-hint: 'Optional package.json to update'
---

# Node version from package.json

`package.json` `engines.node` is the single source of truth for the Node version. Workflows read it with `node-version-file`; nothing else repeats the number.

## Pick the version

- Default: the newest LTS major. Look it up, don't guess:
  `curl -s https://nodejs.org/dist/index.json | jq -r '[.[] | select(.lts != false)][0].version'`
- Firebase Cloud Functions: a `package.json` whose code deploys to Cloud Functions (usually `functions/package.json`, or whatever `firebase.json` `functions.source` points at) uses the newest Node major that Firebase supports instead. Check the "Set Node.js version" section of https://firebase.google.com/docs/functions/manage-functions. Firebase only accepts a bare major here (`"22"`), not a range.
  If `firebase.json` sets `functions.runtime` (`nodejs22`), it wins over `engines`; keep the two in agreement or remove `runtime`.
- Other `package.json` files in a Firebase repo (Hosting site, tooling at the root) still use the newest LTS.

## package.json

```json
"engines": {
  "node": "24"
}
```

- Apps and sites: bare major (`"24"`).
- Packages published to npm: `">=24"` so consumers on newer Node don't get engine warnings.
- Keep `@types/node` on the same major when it's a dependency.

## Workflows

In every `.github/workflows/*.yml` job that uses `actions/setup-node`:

```yaml
- uses: actions/setup-node@v7
  with:
    node-version-file: package.json
```

- Remove `node-version:` (it overrides `node-version-file`) and any `matrix.node` that only exists to pick one version.
- Point at the right file for subdirectory projects: `node-version-file: functions/package.json`. Add `cache-dependency-path` for that directory's lockfile when caching is used.
- Keep the setup-node major the repo already uses if it's current; otherwise use the latest (`gh release view --repo actions/setup-node --json tagName -q .tagName`).

## Clean up

- Delete `.nvmrc`, `.node-version`, and `.tool-versions` node entries that duplicate `engines`, after checking nothing else reads them (Netlify, Vercel, Dockerfiles, scripts). If something does, ask before deleting.
- Search docs (README, CONTRIBUTING) for stale "Node N" mentions.
- If workflows changed and the repo has `.github/workflows/actions.lock`, rerun `gh actions-lock` (see the `actions-lock` skill).

## Verify

- `grep -rn "node-version:" .github/workflows` returns nothing (matrices for intentionally testing several versions are the exception).
- `npm install` still succeeds, then run the project's build/test scripts.
