---
name: flutter-version
description: 'Set the Flutter version in pubspec.yaml environment.flutter and make GitHub Actions install it with flutter-actions/setup-flutter and its version-file action. Use when a Flutter project has workflows, when upgrading Flutter, when workflows hard-code a Flutter version or use subosito/flutter-action, or when removing .fvmrc/.tool-versions duplication.'
license: MIT
argument-hint: 'Optional pubspec.yaml to update'
---

# Flutter version from pubspec.yaml

`pubspec.yaml` `environment.flutter` is the single source of truth for the Flutter version. Workflows install Flutter with [`flutter-actions/setup-flutter`](https://github.com/flutter-actions/setup-flutter) and read the version with its `version-file` action; nothing else repeats the number.

A project is Flutter when its `pubspec.yaml` depends on `flutter: sdk: flutter`. Pure Dart packages keep `dart-lang/setup-dart`.

## Pick the version

- Default: the newest stable release. Look it up, don't guess:
  `curl -s https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json | jq -r '.current_release.stable as $h | .releases[] | select(.hash==$h) | .version'`
- Keep `environment.sdk` (Dart) compatible: its lower bound must not be newer than the Dart version bundled with that Flutter release (`dart_sdk_version` in the same JSON).

## pubspec.yaml

```yaml
environment:
  sdk: ^3.14.0
  flutter: '>=3.50.0'
```

- `version-file` installs the lower bound of the range (`^3.50.0` and `'>=3.50.0 <4.0.0'` both install `3.50.0`; `any` installs `latest`), so CI runs the oldest Flutter the project claims to support. Raise the lower bound to upgrade.
- Apps: set the lower bound to the version you build with. Packages: set it to the oldest version you support.

## Workflows

In every `.github/workflows/*.yml` job that needs Flutter:

```yaml
- id: flutter-version
  uses: flutter-actions/setup-flutter/version-file@v5.0
- uses: flutter-actions/setup-flutter@v5.0
  with:
    version: ${{ steps.flutter-version.outputs.version }}
    cache: true
```

- Replace `subosito/flutter-action` and any other Flutter setup (cloning the SDK, `fvm` installs) with these two steps. Drop `dart-lang/setup-dart` from jobs that also set up Flutter; Flutter bundles Dart.
- Jobs that only need Dart (`dart format --set-exit-if-changed`, Dart-only scripts or subpackages) may use `dart-lang/setup-dart` instead, which is faster. Resolving the Flutter package's dependencies (`pub get`, `analyze`, `test`) still needs Flutter.
- Remove hard-coded Flutter versions (`version:`, `flutter-version:`, `env` like `FLUTTER_VERSION`) and any `matrix.flutter` that only exists to pick one version.
- Subdirectory projects: set `path: app/pubspec.yaml` on the `version-file` step.
- Keep the setup-flutter tag the repo already uses if it's current; otherwise use the latest (`gh release view --repo flutter-actions/setup-flutter --json tagName -q .tagName`). It only publishes exact tags (`v5.0`), not a floating `v5`.
- `channel` defaults to `stable`; only set it for `beta`.
- Publishing to pub.dev adds `flutter-actions/setup-pubdev-credentials` after setup-flutter (see the `pub-publish` skill).

## Clean up

- Delete `.fvmrc`, `.fvm/fvm_config.json`, and `.tool-versions` flutter entries that duplicate `environment.flutter`, after checking nothing else reads them (Codemagic, Dockerfiles, scripts). If something does, ask before deleting.
- Search docs (README, CONTRIBUTING) for stale "Flutter N" mentions.
- If workflows changed and the repo has `.github/workflows/actions.lock`, rerun `gh actions-lock` (see the `actions-lock` skill).

## Verify

- `grep -rnE "subosito/flutter-action|flutter-version:|FLUTTER_VERSION" .github/workflows` returns nothing.
- Every `flutter-actions/setup-flutter@` step gets `version` from a `version-file` step in the same job.
- `flutter pub get` succeeds with the chosen version, then run `flutter analyze` and `flutter test`.
