---
name: pub-publish
description: 'Publish Dart and Flutter packages to pub.dev from GitHub Actions with automated publishing (OIDC) instead of stored credentials. Use when a pubspec.yaml package is published to pub.dev, when adding a publish or release workflow for a Dart package, or when migrating off PUB_CREDENTIALS / credentials.json secrets.'
argument-hint: 'Optional workflow file to update'
---

# Publish to pub.dev with OIDC

Packages publish from `.github/workflows/publish.yml` on a version tag, authenticating with a GitHub OIDC token (`id-token: write`) that `dart-lang/setup-dart` hands to `dart pub`. No stored pub credentials. Based on https://github.com/abraham/c2pa_dart/blob/main/.github/workflows/publish.yml; pub.dev docs: https://dart.dev/tools/pub/automated-publishing.

## Procedure

1. Check `pubspec.yaml`: no `publish_to: none`, has `name`, `version`, and `repository` pointing at this GitHub repo. `CHANGELOG.md` has an entry for the version.
2. Create or update `.github/workflows/publish.yml`:

   ```yaml
   name: Publish

   on:
     push:
       tags:
         # Must match the tag pattern on pub.dev: v{{version}}
         - 'v[0-9]+.[0-9]+.[0-9]+*'

   permissions:
     contents: read

   env:
     # Keep in sync with .github/workflows/ci.yml.
     DART_SDK_VERSION: 3.13.4

   jobs:
     publish:
       runs-on: ubuntu-latest
       environment:
         name: pub.dev
         url: https://pub.dev/packages/<name>/versions/${{ steps.version.outputs.version }}
       concurrency:
         group: deploy-pub.dev
         cancel-in-progress: false
       permissions:
         contents: read
         id-token: write
       steps:
         - uses: actions/checkout@v7
           with:
             persist-credentials: false
         - uses: dart-lang/setup-dart@v1
           with:
             sdk: ${{ env.DART_SDK_VERSION }}
         - id: version
           run: |
             version=$(sed -n 's/^version: *//p' pubspec.yaml)
             test "v$version" = "$GITHUB_REF_NAME"
             echo "version=$version" >> "$GITHUB_OUTPUT"
         - run: dart pub get
         - run: dart pub publish --force
   ```

   - Replace `<name>` with the package name, and the SDK version with the one CI uses (add the same `env` to CI if it hardcodes a version).
   - pub.dev only accepts publishes from runs triggered by a tag that matches the configured pattern. Add `workflow_dispatch` only if the package's pub.dev Admin page allows it (c2pa_dart does).
   - Flutter packages: keep `dart-lang/setup-dart` (it sets up the pub.dev token), add `subosito/flutter-action` after it, and use `flutter pub get` / `flutter pub publish --force`.
   - Several packages in one repo: use a tag pattern per package (`<name>-v{{version}}`) with matching `on.push.tags`, a `working-directory` per package, and publish in dependency order.

3. Remove old credential handling: steps that write `~/.config/dart/pub-credentials.json` or `credentials.json` from a secret, `PUB_CREDENTIALS`/`PUB_TOKEN` env, and third-party publish actions that need them.
4. Follow the other conventions for the new workflow: `environment-url`, `deploy-concurrency`, `workflow-permissions`, then rerun `gh actions-lock`.

## Needs the user

Don't do these yourself; tell the user:

- A brand-new package must publish its first version manually with `dart pub publish`; automated publishing only works for existing packages.
- On `pub.dev/packages/<name>/admin`, Automated publishing: enable publishing from GitHub Actions with this repository and tag pattern `v{{version}}`, and turn on "Require GitHub Actions environment" with `pub.dev`.
- Restrict who can deploy to the `pub.dev` environment (required reviewers or tag rulesets), and delete old pub credential secrets after a successful publish.
- Release by bumping `version` in `pubspec.yaml`, merging, then `git tag v<version> && git push origin v<version>`.

## Verify

- `dart pub publish --dry-run` passes locally.
- The tag pattern in `on.push.tags` matches the one on pub.dev.
