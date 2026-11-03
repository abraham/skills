---
name: prettier
description: 'Set up Prettier formatting in a project that has a package.json: pinned prettier devDependency, .prettierrc.json, .prettierignore, format and format:check scripts, and a format check in the CI lint job. Use when a repo with a package.json has no Prettier, formatting is inconsistent, or the lint job does not check formatting. Skip repos without a package.json.'
---

# Prettier

All files Prettier understands are formatted with it, and CI fails on unformatted files.

Only for repos that already have a `package.json`. If there is none, stop: don't create one just to add Prettier.

## Procedure

1. Install, pinned to an exact version: `npm install --save-dev --save-exact prettier`.
2. `.prettierrc.json`, unless the repo already has a Prettier config (keep the existing one and only migrate it to `.prettierrc.json`):

   ```json
   {
     "singleQuote": true,
     "printWidth": 120
   }
   ```

3. `.prettierignore`. Prettier 3 already skips `.gitignore` entries and `node_modules`, so list only generated files that are committed:

   ```
   package-lock.json
   ```

   Add others such as committed build output, `CHANGELOG.md` written by a release tool, or vendored files.

4. Scripts in `package.json`:

   ```json
   "format": "prettier --write .",
   "format:check": "prettier --check ."
   ```

   Rename existing equivalents (`prettier`, `lint:format`) to these names and update their callers.

5. Add the check to the CI `lint` job (create the job if missing; see the `actions-lock` skill for its shape):

   ```yaml
   - uses: actions/setup-node@v7
     with:
       node-version-file: package.json
   - run: npm ci
   - run: npm run format:check
   ```

6. Run `npm run format` and keep the reformat separate from behavior changes (its own commit, or tell the user). If ESLint is present and has formatting rules, add `eslint-config-prettier` so they don't fight.
7. Rerun `gh actions-lock` if workflows changed and the repo has `.github/workflows/actions.lock`.

## Verify

`npm run format:check` passes.
