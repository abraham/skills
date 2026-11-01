---
name: project-conventions
description: 'Audit a project against my standard setup and fix gaps: Node version in package.json with node-version-file, Firebase deploys via abraham/firebase-wif, environment URLs, least-privilege workflow permissions, a single required CI gate job, gh-actions-lock verified in the lint job, Prettier, and npm start running the dev server. Use when setting up a new repo, bringing an old repo up to date, or asked to apply my conventions.'
---

# Project conventions

Check each rule, report which ones already hold, then fix the rest using the linked skill. Skip rules that don't apply (no `package.json`, no workflows, no Firebase).

| Rule                                                                                              | Applies when                                       | Skill                  |
| ------------------------------------------------------------------------------------------------- | -------------------------------------------------- | ---------------------- |
| Node version lives in `package.json` `engines.node`; workflows use `node-version-file`            | Node is used                                       | `node-version`         |
| `engines.node` is the newest LTS, or the newest Firebase-supported major for Cloud Functions code | `package.json` exists                              | `node-version`         |
| Firebase deploys authenticate with `abraham/firebase-wif`                                         | `firebase.json` or `.firebaserc` exists            | `firebase-deploy`      |
| Jobs with `environment` set `environment.url` to the deployed app                                 | Workflows set `environment`                        | `environment-url`      |
| Every workflow declares least-privilege `permissions`                                             | `.github/workflows` exists                         | `workflow-permissions` |
| One `<file>-required` job (`ci-required`) needs every other CI job; the ruleset requires only it  | CI has more than one job that should block merging | `ci-gate`              |
| Files are formatted with Prettier, checked in the `lint` job                                      | Always                                             | `prettier`             |
| `npm start` starts the dev server                                                                 | `package.json` exists                              | `start-script`         |
| Workflows are locked with `gh-actions-lock`, verified in the `lint` job                           | `.github/workflows` exists                         | `actions-lock`         |

## Order

1. `node-version`, `start-script`, `prettier` (package.json changes).
2. `firebase-deploy`, `environment-url`, `workflow-permissions`, `ci-gate` (workflow changes).
3. `actions-lock` last, since it locks every `uses:` the earlier steps added.
4. `npm run format` so new and edited files are formatted.

## Report

Summarize per rule: already OK, fixed (files changed), or needs the user (for example running `npx @abraham/firebase-wif setup`, deleting old secrets, choosing a production URL, or updating ruleset required checks).
