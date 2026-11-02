---
name: node-test
description: 'Test small Node.js libraries with the built-in node:test runner and node:assert instead of Jest or Vitest. Use when a small npm library uses Jest or Vitest, when adding tests to a Node library, or when trimming test devDependencies.'
argument-hint: 'Optional package.json to update'
---

# Small Node libraries use node:test

Small Node libraries test with `node --test`, `node:test`, and `node:assert/strict`. No Jest, Vitest, or their configs.

## Applies when

The package is a small library published to npm that runs on Node, and its tests:

- don't need a DOM (jsdom, happy-dom) or a browser (Vitest browser mode, Playwright component tests),
- don't rely on module mocking (`jest.mock`, `vi.mock`) beyond a few cases that can become dependency injection,
- don't need Vite plugins or Babel transforms,
- are JavaScript, or TypeScript with only erasable syntax (no `enum`, `namespace`, or parameter properties) so Node's type stripping runs it.

If any of these fail, keep the existing runner and tell the user why.

## package.json

```json
"scripts": {
  "test": "node --test"
}
```

- With no arguments, `node --test` runs `**/*.test.{js,mjs,cjs,ts,mts,cts}`, `**/*-test.*`, `**/*_test.*`, and files under `test/`, skipping `node_modules`. If that picks up extra files (built `dist/` copies), pass a quoted glob so Node expands it: `node --test 'src/**/*.test.ts'`.
- Replace a `test:watch` script, if one exists, with `node --test --watch`.
- Replace coverage scripts, if they exist, with `node --test --experimental-test-coverage`, keeping any thresholds as `--test-coverage-lines=N` (and `-branches`, `-functions`).
- If tests run against built output, run the build first: `"test": "npm run build && node --test"`.

## Convert tests

```js
import { describe, it, beforeEach, mock } from 'node:test';
import assert from 'node:assert/strict';
```

| Jest / Vitest                              | node:test                                                                |
| ------------------------------------------ | ------------------------------------------------------------------------ |
| `test`, `describe`, `it`, `beforeEach`     | Same names from `node:test`; `it.only` needs `--test-only`               |
| `expect(a).toBe(b)`                        | `assert.equal(a, b)`                                                     |
| `expect(a).toEqual(b)`, `toStrictEqual`    | `assert.deepEqual(a, b)` (also compares `undefined` properties)          |
| `expect(a).toBeTruthy()`                   | `assert.ok(a)`                                                           |
| `expect(s).toMatch(/re/)`                  | `assert.match(s, /re/)`                                                  |
| `expect(fn).toThrow(/msg/)`                | `assert.throws(fn, /msg/)`                                               |
| `await expect(p).rejects.toThrow(/msg/)`   | `await assert.rejects(p, /msg/)`                                         |
| `jest.fn()`, `vi.fn()`                     | `mock.fn()`; calls are `fn.mock.calls[i].arguments`, `callCount()`       |
| `jest.spyOn(obj, 'm')`, `vi.spyOn`         | `mock.method(obj, 'm')`                                                  |
| `jest.useFakeTimers()`, `advanceTimersBy…` | `mock.timers.enable({ apis: ['setTimeout'] })`, `mock.timers.tick`       |
| `restoreAllMocks` in `afterEach`           | `mock.restoreAll()`                                                      |
| `toMatchSnapshot()`                        | `(t) => t.assert.snapshot(value)`; update with `--test-update-snapshots` |

- Import from `node:test` explicitly; there are no globals.
- Relative imports in TypeScript tests use the `.ts` extension; enable `allowImportingTsExtensions` or `rewriteRelativeImportExtensions` in `tsconfig.json` if `tsc` complains.
- Delete old snapshot files (`__snapshots__/`) once the new ones are written.

## Clean up

- Remove `jest`, `vitest`, `ts-jest`, `babel-jest`, `@types/jest`, `@vitest/*`, and `eslint-plugin-jest` / `@vitest/eslint-plugin` from devDependencies.
- Delete `jest.config.*`, `vitest.config.*`, Jest config in `package.json`, and Babel config that only existed for Jest.
- Remove `jest` / `vitest/globals` from `tsconfig.json` `types` and test globals from ESLint config.
- CI keeps running `npm test` (see the `node-version` skill's Libraries section for the LTS matrix).

## Verify

- `npm test` passes and reports the same number of tests as before the conversion.
- `grep -rn "jest\|vitest" --include=*.json --include=*.js --include=*.ts --include=*.mjs . --exclude-dir=node_modules` returns nothing.
