---
name: devpilot
description: Doc-driven project development copilot. Use when the user types /devpilot (or asks to start/plan a project with structured docs) — runs a guided interview for new projects, reverse-documents existing codebases, plans work as resumable execution phases, handles feature additions via impact analysis (implement now vs backlog), and generates formal PRD/SRS/gantt documents on demand from a docs/core/ source of truth.
---

# devpilot — Doc-Driven Project Development

devpilot anchors all project work to a two-tier markdown documentation set so that
development stays consistent across sessions and survives interruptions:

- **`docs/core/`** — the single source of truth. Updated on every change. All
  planning and execution decisions come from these files, never from
  conversation memory.
- **`docs/formal/`** — presentation documents (PRD, SRS, gantt). Generated on
  demand from core, never hand-maintained, safe to delete and regenerate.

Document templates live in this skill's `references/templates/` directory
(referred to below as `<templates>`). Read the relevant template file before
generating each document and follow its structure.

## Dispatch

Parse the argument after `/devpilot`:

| Argument | Action |
|---|---|
| *(empty)* | Context detection (below) |
| `init` | New-project interview → generate docs → phase plan |
| `onboard` | Analyze existing codebase → generate docs |
| `feature "<description>"` | Impact analysis → implement now or backlog |
| `resume` | Continue interrupted phase execution |
| `docs <type>` | Generate formal doc: `prd`, `srs`, `gantt`, or `all` |
| `status` | Show phase/task progress summary |

## Locating the docs folder

The docs folder is `docs/` by default but the name is customizable and recorded
in the config file. To locate it: check `docs/core/config.md` first; if absent,
glob for `*/core/config.md` and accept a file containing the marker
`<!-- devpilot-config -->`. When found, read `config.md` fully before doing
anything else — it defines the document language, docs folder name, divisions,
and which core docs are active.

## Context detection (bare `/devpilot`)

1. Locate the config file (above).
2. **No config found:**
   - Folder is empty or has no source files → suggest `init`.
   - Folder has source code → suggest `onboard`.
   - Confirm with the user (one question) before proceeding.
3. **Config found:**
   - `phases.md` has unchecked tasks → offer `resume`, showing the next pending task.
   - Otherwise → show `status` output and list available actions.

## Global rules

- **Interviews ask ONE question at a time.** Wait for each answer. Prefer
  questions with narrow answer spaces; use AskUserQuestion when options are
  enumerable. Never dump a list of questions.
- **Language:** every generated document is written in the language recorded in
  `config.md`. The language is asked once, during init/onboard.
- **Never overwrite user content** in existing `CLAUDE.md`/`AGENTS.md` — append
  a clearly marked section instead:
  `<!-- devpilot:start -->` … `<!-- devpilot:end -->`.
- **Docs are navigation, not substitutes:** before editing any source file
  during execution, Read the actual current file first.
- **Only generate applicable core docs.** No API → skip `api-contract.md`; no
  database → skip `database.md`. Record the active-docs list (and why any were
  skipped) in `config.md`. Never emit empty shell documents.
- **Estimates are estimates.** Mandays and timeline figures are planning
  figures ("as if executed by a team per division"), not commitments. Templates
  label them as such — keep that labeling in generated output.
- User edits to core docs are authoritative. If a manual edit contradicts
  `phases.md` state, surface the conflict and ask before proceeding.

## /devpilot init — new project

1. **Preflight:** if the target docs folder already exists with non-devpilot
   content, ask: merge into it, use a different folder name (recorded in
   config), or abort. If the folder already contains substantial source code,
   suggest `onboard` instead and confirm.
2. **Interview** (one question at a time), covering at minimum:
   1. Project goal, target users, and the problem being solved.
   2. Document language (Indonesian / English / other).
   3. Feature scope — probe until a concrete, enumerable module/feature list emerges.
   4. Constraints: deadline, budget, required integrations, things it must not do.
   5. Tech stack preference — if the user has none, propose a stack that fits
      the project with reasoning, and confirm.
   6. Project type and team divisions — auto-propose divisions from project
      type (web app → Frontend, Backend, UI/UX, QA, DevOps; mobile app → those
      + Mobile; API/CLI-only → drop UI/UX), then let the user confirm or edit.

   Stop interviewing as soon as every applicable core doc can be written
   concretely — don't pad.
3. **Generate `docs/core/`** from `<templates>` (core-*.md): `config`,
   `requirements`, `features` (per-division mandays for every feature),
   `tech-stack`, `architecture`, `database` (if applicable), `api-contract`
   (if applicable), `phases` (ordered, dependency-aware plan; phases and tasks
   numbered `N` / `N.M`), and `backlog` (empty scaffold).
4. **Production hygiene** (section below).
5. **Git:** if the folder is not a git repository, offer `git init`.
6. Report the generated files, then ask whether to start executing Phase 1 now.

## /devpilot onboard — existing project

1. **Preflight:** same docs-folder conflict handling as init step 1.
2. **Analyze the codebase:** structure, stack (read manifests — `package.json`,
   `composer.json`, `pyproject.toml`, `go.mod`, etc.), entry points, database
   schema (migrations/models), API routes, and observable behavior.
   - If `graphify-out/graph.json` exists or the `graphify` CLI is available,
     use `graphify query` / `graphify explain` to accelerate and deepen the
     analysis. This is an **optional** enhancement — onboard must work
     identically well without graphify, with no confusing degradation.
   - **Large codebase:** analyze incrementally — entry points, schema, and
     routes first. In the generated docs, explicitly mark which areas were
     fully analyzed vs sampled. Recommend running graphify when the codebase
     exceeds comfortable direct analysis.
3. **Short interview:** document language; confirmation of detected
   stack/architecture; divisions (auto-propose + confirm); and anything the
   code cannot reveal (goals, target users).
4. **Generate `docs/core/`** describing **what already exists**, so future
   feature work does not conflict with current behavior. `phases.md` contains a
   single completed marker — `Phase 0 — Existing system (baseline)` — and no
   pending phases.
5. **Production hygiene** (section below).
6. Report the generated files and suggest next actions (`feature`, `docs`).

## /devpilot feature "<description>"

1. If no core docs exist → explain that docs are required first and route to
   `onboard` (or `init` if the folder is empty). Do not proceed.
2. Run a mini-interview **only if** the description is ambiguous.
3. **Impact analysis:** read the core docs and produce a table — affected doc →
   section → change summary. Examples: new table → `database.md` (+ migration
   note); new endpoints → `api-contract.md`; mandays delta → `features.md`;
   requirement changes → `requirements.md`; and the new phases that
   implementation would need.
4. Ask exactly **one** decision question: implement **now** or **later**?
5. - **Now** → apply the updates to every affected core doc → append the new
     phase(s) with tasks to `phases.md` → begin Phase execution.
   - **Later** → append an entry to `backlog.md`: date, description, the full
     impact analysis, status `deferred`. Touch **no other document**.
6. **Backlog promotion** (user asks to implement a backlog item, via `feature`
   or plain request): first re-validate the saved impact analysis against the
   **current** core docs — they may have changed since deferral. If the
   analysis is stale, regenerate it and show the user what changed before
   executing. Then proceed as "now".

## Phase execution

- Work through phases strictly in order, **auto-continuing to the next phase
  without waiting for approval**. The user can interrupt at any time.
- Update the task checkbox in `phases.md` **immediately** when each task
  completes — never batch updates to the end of a phase.
- When a phase completes: update its status in `phases.md`, then create a git
  commit checkpoint in conventional-commit format referencing the phase, e.g.
  `feat: phase 3 — auth module` (use `fix:`/`chore:` where more accurate).
- Give a brief one-or-two-line report per completed phase, then continue.
- **Not a git repo:** offer `git init`. If declined, execute without commit
  checkpoints and warn that resume granularity falls back to `phases.md`
  checkboxes only.

### Code quality standards

Every task's implementation is written to senior-developer standards:

- **Best practices & conventions:** follow the established best practices of
  the project's language/framework and the conventions already present in the
  codebase — match its style, don't introduce a parallel one.
- **Security-aware by default:** validate all external input at system
  boundaries; parameterized queries only (never string-concatenated SQL); no
  hardcoded secrets or credentials — use environment variables; escape/sanitize
  output rendered to users (XSS); enforce authorization on every endpoint
  (least privilege); never leak sensitive data in error messages or logs.
- **Explicit error handling:** never silently swallow errors; user-facing
  messages stay friendly, server-side logs carry the detail.
- **Readable and maintainable:** descriptive naming, small focused functions,
  early returns over deep nesting, named constants over magic numbers, no
  speculative abstractions (KISS/DRY/YAGNI).
- **Tests:** implement the test tasks the phase plan defines; when a phase
  touches critical paths that lack tests, flag it and recommend adding a test
  task rather than skipping silently.

## /devpilot resume

1. Read `phases.md` → find the first unchecked task.
2. Run `git status` and `git diff` → is there uncommitted partial work?
3. Inspect any half-finished files, then complete (or cleanly redo) **only the
   interrupted task**. Never redo completed phases.
4. Continue normal phase execution from there.
5. Nothing unchecked → report that all phases are complete and suggest
   `/devpilot status` or `/devpilot feature`.

## /devpilot docs <type>

- Types: `prd`, `srs`, `gantt`, or `all`.
- If no core docs exist → refuse, pointing to `init`/`onboard`.
- Generate into `docs/formal/` from `<templates>` (formal-*.md), derived
  **entirely from the current `docs/core/` contents** — never from memory of
  past sessions.
- Every formal doc carries the generated-notice header from its template,
  filled with the generation date. Overwrite any previous version — formal
  docs are disposable outputs.
- `gantt`: mermaid gantt chart with one section per division, planned
  dependency-aware with divisions working in parallel where dependencies allow.

## /devpilot status

Show: per-phase progress (done/total tasks), the current or next pending task,
backlog item count, the active-docs list from `config.md`, and the most recent
checkpoint commit (`git log --oneline` grep for phase commits).

## Production hygiene

During init and onboard, ensure AI/doc paths are excluded from production
builds — **at build/deploy level, never via `.gitignore`** (these files stay
committed on all branches, including main):

1. Paths to exclude: the docs folder (actual configured name), `CLAUDE.md`,
   `CLAUDE.local.md`, `AGENTS.md`, `.claude/`, plus equivalent agent files
   present in the project (`GEMINI.md`, `.cursor/`, etc.).
2. Detect the stack and touch only the relevant exclusion files:
   `Dockerfile` present → `.dockerignore`; Vercel project → `.vercelignore`;
   Google Cloud deploy → `.gcloudignore`; publishable npm package →
   `.npmignore`. Create the file if missing; if it exists, append a marked
   block (`# devpilot:start` … `# devpilot:end`) preserving existing content.

## Templates

`<templates>` contains one template per document:

- Core: `core-config.md`, `core-requirements.md`, `core-features.md`,
  `core-tech-stack.md`, `core-architecture.md`, `core-database.md`,
  `core-api-contract.md`, `core-phases.md`, `core-backlog.md`
- Formal: `formal-prd.md`, `formal-srs.md`, `formal-timeline-gantt.md`

Read the template before generating each document. Follow its structure, fill
every `{placeholder}`, translate headings into the configured document
language, and strip the instructional HTML comments from the generated output
(except the `<!-- devpilot-config -->` marker, which must be kept).
