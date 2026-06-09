# Testing Patterns

**Analysis Date:** 2026-06-09

## Repository Type

This is a **content-only skill extension**. There are no TypeScript/JavaScript source files, no test files, and no test runner configured. Testing is handled structurally through CI gates and package shape validation rather than unit/integration test suites.

## Test Framework

**Runner:**
- Not applicable — no test files exist in the repository

**Assertion Library:**
- Not applicable

**Run Commands:**
```bash
# CI runs this; exits 0 silently when no test script is defined
corepack pnpm test --if-present
```

The `--if-present` flag means CI does not fail when `"test"` script is absent from `package.json`.

## CI-Based Quality Gates (replaces test suite)

All quality validation is done in `.github/workflows/ci.yml` via two jobs:

**`build` job gates:**
1. **First-party dependency shape check** — inline Node.js script validates that no `@cinatra-ai/*` or `@cinatra/*` packages leaked into `dependencies`/`devDependencies`/`optionalDependencies`. Any first-party peer must be declared in `peerDependencies` with `peerDependenciesMeta[pkg].optional = true`.
2. **Typecheck** — skipped for source mirrors (repos with host-internal peers). For standalone repos, runs `tsc --noEmit` using a local or ephemeral TypeScript install.
3. **Pack dry run** — `npm pack --dry-run` validates the package shape and publish payload without resolving peers.

**`kind-gates` job:**
- Runs after `build` (via `needs: build`)
- For `kind: "skill"` extensions, no extra gate is applied (placeholder step echoes "No kind-specific gate for this extension kind.")
- Workflow/agent kinds would have additional `extension-kind-gate.mjs` steps appended by the extraction script.

## SKILL.md Validation

No automated content validation exists for `skills/drupal-widget-chat/SKILL.md`. The file is validated by:
- Human review of the YAML frontmatter (`name`, `description` fields)
- Behavioral review of the system prompt rules
- Integration testing in the Cinatra monorepo (not in this repo)

## Test File Organization

**Location:**
- No test files exist

**Naming:**
- Not applicable

## Mocking

- Not applicable — no test suite

## Fixtures and Factories

- Not applicable — no test suite

## Coverage

**Requirements:** Not enforced — no test suite exists

## Test Types

**Unit Tests:**
- Not present

**Integration Tests:**
- Run by the Cinatra monorepo when this repo is cloned in as a workspace dependency. Not runnable standalone.

**E2E Tests:**
- Not applicable

## Adding Tests in the Future

If TypeScript sources are added under `src/` (as anticipated by `tsconfig.json`):

1. Add a test runner to `devDependencies` (e.g., `vitest` — compatible with ES module + bundler module resolution already configured)
2. Add `"test": "vitest run"` to `package.json` scripts
3. Place test files co-located with sources as `src/**/*.test.ts` or in a `src/__tests__/` directory
4. CI will automatically pick up the `test` script via `pnpm test --if-present`

Note: If any `@cinatra-ai/*` peer dependency is added, CI will skip standalone tests — the monorepo owns the test run in that case.

---

*Testing analysis: 2026-06-09*
