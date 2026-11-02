---
name: project-conventions
description: 'Audit a project against my standard setup and fix gaps: Node version in package.json with node-version-file, Firebase deploys via abraham/firebase-wif, npm publishing with OIDC trusted publishing, pub.dev publishing with OIDC, environment URLs, one deploy at a time via concurrency, least-privilege workflow permissions, a single required CI gate job, gh-actions-lock verified in the lint job, Prettier, and npm start running the dev server. Use when setting up a new repo, bringing an old repo up to date, or asked to apply my conventions.'
---

# Project conventions

Check each rule, report which ones already hold, then fix the rest using the linked skill. Skip rules that don't apply (no `package.json`, no workflows, no Firebase).

| Rule                                                                                              | Applies when                                        | Skill                  |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------- | ---------------------- |
| Node version lives in `package.json` `engines.node`; workflows use `node-version-file`            | Node is used                                        | `node-version`         |
| `engines.node` is the newest LTS, or the newest Firebase-supported major for Cloud Functions code | `package.json` exists                               | `node-version`         |
| Libraries set `engines.node` to `>=` the oldest supported LTS and test every supported LTS major  | `package.json` is published to npm (not `private`)  | `node-version`         |
| Firebase deploys authenticate with `abraham/firebase-wif`                                         | `firebase.json` or `.firebaserc` exists             | `firebase-deploy`      |
| npm packages publish with trusted publishing (OIDC), no `NPM_TOKEN`                               | `package.json` is published to npm (not `private`)  | `npm-publish`          |
| Dart packages publish to pub.dev with automated publishing (OIDC)                                 | `pubspec.yaml` is published (no `publish_to: none`) | `pub-publish`          |
| Jobs with `environment` set `environment.url` to the deployed app                                 | Workflows set `environment`                         | `environment-url`      |
| Deploy jobs use a `concurrency` group per target so only one runs at a time                       | Workflows deploy                                    | `deploy-concurrency`   |
| Every workflow declares least-privilege `permissions`                                             | `.github/workflows` exists                          | `workflow-permissions` |
| One `<file>-required` job (`ci-required`) needs every other CI job; the ruleset requires only it  | CI has more than one job that should block merging  | `ci-gate`              |
| Files are formatted with Prettier, checked in the `lint` job                                      | Always                                              | `prettier`             |
| `npm start` starts the dev server                                                                 | `package.json` exists                               | `start-script`         |
| Workflows are locked with `gh-actions-lock`, verified in the `lint` job                           | `.github/workflows` exists                          | `actions-lock`         |

## Order

1. `node-version`, `start-script`, `prettier` (package.json changes).
2. `firebase-deploy`, `npm-publish`, `pub-publish`, `environment-url`, `deploy-concurrency`, `workflow-permissions`, `ci-gate` (workflow changes).
3. `actions-lock` last, since it locks every `uses:` the earlier steps added.
4. `npm run format` so new and edited files are formatted.

## Report

Summarize per rule: already OK, fixed (files changed), or needs the user (for example running `npx @abraham/firebase-wif setup`, configuring an npm trusted publisher or pub.dev automated publishing, deleting old secrets, choosing a production URL, or updating ruleset required checks).
