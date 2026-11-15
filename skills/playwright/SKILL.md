---
name: playwright
description: 'Run Playwright in GitHub Actions: install browsers with npx playwright install --with-deps or use the mcr.microsoft.com/playwright image at the installed version, and key any browser cache on the OS the job runs in (from /etc/os-release, including job containers), the architecture, and the Playwright version. Use when a workflow runs Playwright, caches ms-playwright, keys a Playwright cache on runner.os, runs Playwright in a container, or when cached browsers fail to launch.'
license: MIT
argument-hint: 'Optional workflow file to update'
---

# Playwright in GitHub Actions

Every job that runs Playwright gets browsers that match its OS and the installed Playwright version, either from the Playwright image or from `npx playwright install --with-deps`. If the job caches browsers, the cache key includes the OS the steps actually run in.

## Find the jobs

- `grep -rnE "playwright|ms-playwright" .github/workflows` finds jobs that run Playwright, install browsers, or cache them.
- Also check `PLAYWRIGHT_BROWSERS_PATH`, which moves the browser folder; any cache `path` must match it.

## Playwright image

Jobs whose `container:` is `mcr.microsoft.com/playwright` already have the browsers and OS packages:

- Make the image tag match the installed `@playwright/test` version (`mcr.microsoft.com/playwright:v1.65.1-noble`). A mismatch makes Playwright look for browser builds the image doesn't have.
- Delete their browser cache and `playwright install` steps.

## Install browsers

Other jobs install browsers after dependencies (`npm ci`):

```yaml
- run: npx playwright install --with-deps
```

- `--with-deps` installs the OS packages the browsers need. Drop it only where the container already has them or the user can't `apt-get` (non-root containers).
- Install only the browsers the job tests (`npx playwright install --with-deps chromium`) when the config uses fewer than all of them.

## Cache browsers

Caching is optional; restoring the cache takes about as long as downloading on GitHub-hosted runners. If a job caches browsers, key the cache on the OS read inside the job.

Playwright downloads browser builds for a specific OS release (Chromium and WebKit for Ubuntu 22.04 and 24.04 are different builds) and architecture. `runner.os` is just `Linux` for every Ubuntu runner and every job container, and `ImageOS`/`matrix.os` describe the host, not the `container:` the steps run in. A key built from those restores browsers built for another OS, and Playwright skips the download because the browser folder already exists, so tests fail to launch or crash.

Replace each Playwright cache with these steps, after `npm ci`:

```yaml
- id: playwright
  shell: bash
  run: |
    if [ -f /etc/os-release ]; then
      . /etc/os-release
      echo "os=${ID}-${VERSION_ID}" >> "$GITHUB_OUTPUT"
    else
      echo "os=${ImageOS:-$RUNNER_OS}" >> "$GITHUB_OUTPUT"
    fi
    echo "version=$(npx playwright --version | awk '{print $2}')" >> "$GITHUB_OUTPUT"
- uses: actions/cache@v6.1.0
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ steps.playwright.outputs.os }}-${{ runner.arch }}-${{ steps.playwright.outputs.version }}
- run: npx playwright install --with-deps
```

- The OS step runs inside the job container when there is one, so `/etc/os-release` gives the container's distro (`debian-12`, `ubuntu-24.04`). macOS and Windows runners fall back to `ImageOS` (`macos15`, `win25`).
- `runner.arch` separates `ubuntu-24.04` from `ubuntu-24.04-arm`.
- Read the version from the installed package, not `package.json`, so a `^` range can't key two versions the same.
- Don't add `restore-keys`. A partial match restores another OS's or version's browsers, and the saved cache keeps growing with old builds.
- Always run `npx playwright install --with-deps`, not only on a cache miss. It skips browsers that are already present, downloads any that are missing, and installs the OS packages, which the cache doesn't contain.
- If jobs install different browsers, add them to the key (`...-chromium`) so one job's cache doesn't hide the other's missing browsers.
- `path` per OS: Linux `~/.cache/ms-playwright`, macOS `~/Library/Caches/ms-playwright`, Windows `~\AppData\Local\ms-playwright`, or the value of `PLAYWRIGHT_BROWSERS_PATH` if set. In a matrix across OSes, list all three paths.
- Keep the `actions/cache` tag the repo already uses if it's current; otherwise use the latest (`gh release view --repo actions/cache --json tagName -q .tagName`).

## Clean up

- If workflows changed and the repo has `.github/workflows/actions.lock`, rerun `gh actions-lock` (see the `actions-lock` skill).

## Verify

- Jobs in the Playwright image use the tag matching the installed Playwright version and don't install or cache browsers.
- Every Playwright cache key includes the `/etc/os-release` output, `runner.arch`, and the Playwright version; none rely on `runner.os`, `ImageOS`, or `matrix.os` alone, and none use `restore-keys`.
- `actionlint` passes on the changed files if it's installed.
- In a run, the OS step prints the container's distro for container jobs, and Playwright tests launch after a cache hit.
