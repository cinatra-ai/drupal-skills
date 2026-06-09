# Technology Stack

**Analysis Date:** 2026-06-09

## Languages

**Primary:**
- TypeScript (ES2023 target) — declared in `tsconfig.json`; no `.ts` sources are currently tracked (content-only extension with SKILL.md as the primary artifact)

**Secondary:**
- Not applicable

## Runtime

**Environment:**
- Node.js 24 (pinned in `.github/workflows/ci.yml`)

**Package Manager:**
- pnpm via Corepack (`corepack enable` in CI)
- Lockfile: not committed (CI runs `--no-frozen-lockfile`); `.npmrc` sets `auto-install-peers=false`

## Frameworks

**Core:**
- None — this is a content-only Cinatra skill extension. The primary deliverable is `skills/drupal-widget-chat/SKILL.md` (a structured system-prompt artifact consumed by the Cinatra platform runtime).

**Testing:**
- None detected (no test files, no test runner configured in `package.json`)

**Build/Dev:**
- TypeScript compiler (`tsc`) — configured via `tsconfig.json` for any future `.ts` sources; currently no TypeScript sources are tracked

## Key Dependencies

**Critical:**
- None declared in `package.json` — `dependencies`, `devDependencies`, and `peerDependencies` are all absent. The `cinatra` manifest block declares `"dependencies": []`.

**Infrastructure:**
- `@cinatra-ai/drupal-skills` is the npm package name (scoped to the `@cinatra-ai` org)
- First-party `@cinatra-ai/*` / `@cinatra/*` packages must be declared as optional `peerDependencies` per CI enforcement (`.github/workflows/ci.yml`); none are currently declared

## Configuration

**Environment:**
- No `.env` files detected
- No runtime environment variables required by this repo directly; environment context (`instanceId`, `nodeId`) is injected server-side by the Cinatra platform at request time

**Build:**
- `tsconfig.json` — standalone strict TypeScript config; targets `src/`, outputs to `dist/`, `ESNext` modules, `bundler` module resolution, JSX enabled (`react-jsx`)
- `.npmrc` — `auto-install-peers=false`

## Platform Requirements

**Development:**
- Node.js 24+, pnpm via Corepack

**Production:**
- Cinatra Marketplace / Cinatra platform runtime
- Published via GitHub Release triggering `.github/workflows/release.yml`, which delegates to the reusable workflow `cinatra-ai/.github/.github/workflows/reusable-extension-release.yml@main`
- Registry target: `registry.cinatra.ai` (marketplace promotion saga, not Verdaccio direct publish)

---

*Stack analysis: 2026-06-09*
