# Coding Conventions

**Analysis Date:** 2026-06-09

## Repository Type

This is a **content-only skill extension** for the Cinatra platform. The repository contains no TypeScript/JavaScript source files (`src/` directory is absent despite being referenced in `tsconfig.json`). All functional content lives in `skills/drupal-widget-chat/SKILL.md`, which is a YAML-frontmatter Markdown file that acts as a system prompt definition.

## Naming Patterns

**Files:**
- Skill definition files are named `SKILL.md` (UPPERCASE) and live at `skills/<skill-name>/SKILL.md`
- Skill directories use kebab-case: `skills/drupal-widget-chat/`
- Workflow files use kebab-case: `.github/workflows/ci.yml`, `.github/workflows/release.yml`

**Skill Names:**
- Skill directory names use kebab-case matching the `name:` field in SKILL.md frontmatter: `drupal-widget-chat`

**Package Naming:**
- npm package uses scoped name with org prefix: `@cinatra-ai/drupal-skills`

## SKILL.md Format

**Structure:**
- YAML frontmatter block (required): contains `name` and `description` fields
- Markdown body: structured with `##` section headings
- Sections use imperative present tense ("You are…", "Call `tool` whenever…")

**Example pattern from `skills/drupal-widget-chat/SKILL.md`:**
```markdown
---
name: drupal-widget-chat
description: System prompt for the Cinatra Drupal in-CMS chat widget. Decides when to call drupal_content_editor_run vs answering conversationally; specifies how to summarize tool results without pasting raw JSON.
---

You are the Cinatra assistant embedded in the Drupal node editor...
```

**Section conventions in SKILL.md:**
- Lead with role statement ("You are the Cinatra assistant embedded in…")
- "## Your job" section: enumerate actions with numbered list and bold action labels
- "## When you call [tool]" sections: give concrete JSON-to-English summarization examples
- "## Errors" section: specify fallback phrasing for error cases
- "## [Edge case]" sections: call out timing/performance warnings (e.g., "## Long edits")
- Hard rules listed under `**Hard rules:**` using bullet points prefixed with NEVER/ALWAYS

## Tool Call Conventions in SKILL.md

- Tool names use snake_case: `drupal_content_editor_run`
- Tool arguments documented explicitly: which args to pass, which to omit
- Examples shown as inline JSON: `{ nodeId: "42", changes: [...] }`
- Summarization templates shown with `→` arrow to indicate the expected reply text

## package.json Conventions

- `"type": "module"` — ES module package
- Cinatra metadata under `"cinatra"` key with `apiVersion`, `kind`, `dependencies` fields
- `kind: "skill"` for skill extensions
- No scripts defined (content-only; CI handles all gates)
- No dependencies, devDependencies, or optionalDependencies (source mirror with no first-party @cinatra-ai peers)

## TypeScript Configuration

- `tsconfig.json` present as a CI contract (required by the Cinatra extraction baseline) even though no TypeScript sources exist yet
- `strict: true` with `noImplicitAny: false` override
- `verbatimModuleSyntax: true`
- `moduleResolution: "bundler"`, `module: "ESNext"`, `target: "ES2023"`
- `rootDir: "src"`, `outDir: "dist"` — reserved for future TypeScript source files

## Code Style

**Formatting:**
- Not applicable — no TypeScript/JavaScript source files exist

**Linting:**
- Not applicable — no TypeScript/JavaScript source files exist

## Import Organization

- Not applicable — no TypeScript/JavaScript source files exist

## Error Handling (in SKILL.md behavior definitions)

- Tool errors: surface with generic user-facing message ("I couldn't apply that edit — please try again, and let me know if it keeps failing.")
- Never expose: internal error messages, stack traces, or HTTP status codes
- Pattern: always give one retry instruction + escalation path

## Comments

**In CI YAML (`.github/workflows/ci.yml`):**
- Block comments at top of file explain the CI strategy
- Inline comments on each job step explain skip rationale
- `# NOTE:` prefix used for architectural notes
- `#` comments explain decision branches before each conditional block

**In release YAML (`.github/workflows/release.yml`):**
- File-level block comment explains the publishing strategy and dormancy condition

## Module Design

- One skill per subdirectory under `skills/`
- Each skill is self-contained: `SKILL.md` is the only required file
- Package exports the skill catalog via `package.json` shape; no JS barrel files needed

---

*Convention analysis: 2026-06-09*
