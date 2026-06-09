# Codebase Structure

**Analysis Date:** 2026-06-09

## Directory Layout

```
drupal-skills/
├── .github/
│   └── workflows/
│       ├── ci.yml          # Push/PR gate: shape check, typecheck, test, dry-pack
│       └── release.yml     # GitHub Release → Cinatra Marketplace publish
├── .planning/
│   └── codebase/           # GSD codebase map documents (this directory)
├── skills/
│   └── drupal-widget-chat/
│       └── SKILL.md        # System prompt defining all widget behavior
├── LICENSE                 # Apache-2.0
├── README.md               # Human-readable overview and capability list
├── package.json            # npm manifest + cinatra extension manifest block
└── tsconfig.json           # TypeScript config (for future src/ sources)
```

## Directory Purposes

**`skills/`:**
- Purpose: Root container for all Cinatra skill definitions in this repo
- Contains: One subdirectory per skill, each with a `SKILL.md`
- Key files: `skills/drupal-widget-chat/SKILL.md`

**`skills/drupal-widget-chat/`:**
- Purpose: The single skill shipped by this repo — the Drupal in-CMS chat widget system prompt
- Contains: `SKILL.md` only (no compiled code)
- Key files: `skills/drupal-widget-chat/SKILL.md`

**`.github/workflows/`:**
- Purpose: GitHub Actions CI/CD pipelines
- Contains: `ci.yml` (build gate), `release.yml` (marketplace publish)
- Key files: `.github/workflows/ci.yml`, `.github/workflows/release.yml`

**`.planning/codebase/`:**
- Purpose: GSD codebase map output — consumed by `/gsd-plan-phase` and `/gsd-execute-phase`
- Contains: ARCHITECTURE.md, STRUCTURE.md (this file)
- Generated: Yes (by GSD mapper agent)
- Committed: Yes

## Key File Locations

**Entry Points:**
- `skills/drupal-widget-chat/SKILL.md`: Primary artifact — the system prompt injected by the Cinatra runtime into every Drupal widget chat session

**Configuration:**
- `package.json`: npm + Cinatra extension manifest (`cinatra.kind = "skill"`, `cinatra.apiVersion`, `cinatra.dependencies`)
- `tsconfig.json`: TypeScript compiler config targeting a future `src/` directory (ES2023, ESNext modules, strict)

**CI/CD:**
- `.github/workflows/ci.yml`: Validates first-party dep shape, runs typecheck/test/dry-pack; handles both source-mirror and standalone repo modes
- `.github/workflows/release.yml`: Thin caller delegating to `cinatra-ai/.github` reusable release workflow

**Documentation:**
- `README.md`: Human-facing capability summary and integration list
- `LICENSE`: Apache-2.0

## Naming Conventions

**Files:**
- Skill definitions: `SKILL.md` (uppercase) inside a named subdirectory under `skills/`
- Workflow files: lowercase with hyphens (`ci.yml`, `release.yml`)
- Config files: standard names (`package.json`, `tsconfig.json`)

**Directories:**
- Skill subdirectories: kebab-case matching the skill name (`drupal-widget-chat`)
- Skills root: `skills/` (plural, lowercase)

## Where to Add New Code

**New skill:**
- Create `skills/<skill-name>/SKILL.md` following the YAML front-matter + Markdown sections pattern of `skills/drupal-widget-chat/SKILL.md`
- If the skill needs a separate npm package, create a new `package.json` with `cinatra.kind = "skill"`

**TypeScript source code (future):**
- Implementation: `src/` (referenced by `tsconfig.json` as `rootDir`)
- Compiled output: `dist/` (referenced by `tsconfig.json` as `outDir`, gitignored)

**Tests:**
- Location: alongside sources in `src/` or in a `tests/` directory (no tests currently exist)
- Runner: determined by CI — runs `pnpm test --if-present`

**Additional workflow gates:**
- Append steps to the `kind-gates` job in `.github/workflows/ci.yml` per the inline documentation comments

## Special Directories

**`dist/`:**
- Purpose: TypeScript compiled output
- Generated: Yes (by `tsc`)
- Committed: No (excluded by `tsconfig.json`; assumed gitignored)

**`.planning/`:**
- Purpose: GSD planning and codebase map artifacts
- Generated: Yes (by GSD agents)
- Committed: Yes

---

*Structure analysis: 2026-06-09*
