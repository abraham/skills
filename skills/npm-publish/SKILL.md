---
name: npm-publish
description: 'Publish npm packages from GitHub Actions with npm trusted publishing (OIDC) instead of an NPM_TOKEN secret. Use when a package.json package is published to npm, when adding a publish or release workflow, or when migrating off NPM_TOKEN / NODE_AUTH_TOKEN.'
argument-hint: 'Optional workflow file to update'
---

# Publish to npm with OIDC

Packages publish from `.github/workflows/publish.yaml` on a version tag, authenticating with npm trusted publishing (`id-token: write`). No `NPM_TOKEN` secret. Based on https://github.com/abraham/pkg-ok/blob/main/.github/workflows/publish.yaml.

## Procedure

1. Check `package.json`: not `"private": true`, has `name`, `version`, and `repository.url` pointing at this GitHub repo (npm checks it against the provenance).
2. Create or update `.github/workflows/publish.yaml`:

   ```yaml
   name: Publish Package

   on:
     push:
       tags:
         - 'v*'

   permissions:
     contents: read

   jobs:
     publish:
       runs-on: ubuntu-latest
       environment:
         name: npm
         url: https://www.npmjs.com/package/<name>/v/${{ steps.version.outputs.version }}
       concurrency:
         group: deploy-npm
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
             node-version: lts/*
         - id: version
           run: |
             version=$(node -p "require('./package.json').version")
             test "v$version" = "$GITHUB_REF_NAME"
             echo "version=$version" >> "$GITHUB_OUTPUT"
         - run: npm ci
         - run: npm run build --if-present
         - run: npm publish
   ```

   - Replace `<name>` with the package name (scoped names stay as is: `@scope/name`).
   - The version step fails the run when the tag doesn't match `package.json`.
   - Publish with the newest LTS (`lts/*`), not the `engines` range; see the `node-version` skill's Libraries section. Trusted publishing needs npm 11.5.1+, which Node 24+ ships.
   - Don't run tests here; CI already passed on the tagged commit. Remove existing test or lint steps from publish workflows.

3. Remove token auth from publish steps: `NODE_AUTH_TOKEN`, `NPM_TOKEN`, `registry-url` added only for token auth, `.npmrc` lines with `_authToken`, and `--provenance` (trusted publishing adds provenance automatically).
4. Follow the other conventions for the new workflow: `node-version`, `environment-url`, `deploy-concurrency`, `workflow-permissions`, then rerun `gh actions-lock`.

## Needs the user

Don't do these yourself; tell the user:

- On npmjs.com, package Settings, Trusted publishing: add GitHub Actions with this repo, workflow file `publish.yaml`, and environment `npm`. A package that isn't on npm yet may need its first version published manually.
- After a successful OIDC publish, set the package to require 2FA and disallow tokens, and delete the `NPM_TOKEN` repository secret.
- Release by pushing a tag: `npm version <patch|minor|major>` then `git push --follow-tags`.

## Verify

- `grep -rn "NPM_TOKEN\|NODE_AUTH_TOKEN" .github` returns nothing.
- `npm publish --dry-run` succeeds locally and lists only the intended files.
