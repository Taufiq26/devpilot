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
| `revise "<description>"` | Change existing behavior → impact analysis → implement now or backlog |
| `fix "<description>"` | Bug fix → root cause + impact → implement now |
| `resume` | Continue interrupted phase execution |
| `docs <type>` | Generate formal doc: `prd`, `srs`, `gantt`, `dashboard`, or `all` |
| `status` | Show phase/task progress summary |

## Locating the docs folder

The docs folder is `docs/` by default but the name is customizable and recorded
in the config file. To locate it: check `docs/core/config.md` first; if absent,
glob for `*/core/*config.md` (config docs generated since v0.2 carry a numeric
prefix, e.g. `00-config.md` — older projects don't) and accept a file
containing the marker `<!-- devpilot-config -->`. When found, read `config.md`
fully before doing anything else — it defines the document language, docs
folder name, divisions, which core docs are active, and whether this
project's core filenames are numbered (config's "Core doc filenames" row).
When referencing a core doc by name in generated prose (impact-analysis
tables, changelog entries, `docs/README.md`), always use its base name
without the numeric prefix (e.g. "database.md") — the actual on-disk filename
may or may not carry one, and readers can pattern-match either way.

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
  database → skip `database.md`; no user interface → skip `design.md`. Record
  the active-docs list (and why any were skipped) in `config.md`. Never emit
  empty shell documents.
- **Estimates are estimates.** Mandays and timeline figures are planning
  figures ("as if executed by a team per division"), not commitments. Templates
  label them as such — keep that labeling in generated output.
- User edits to core docs are authoritative. If a manual edit contradicts
  `phases.md` state, surface the conflict and ask before proceeding.
- **`docs/README.md` is always generated, regardless of who executes next.**
  Not everyone picking up the project will have the devpilot skill loaded —
  the docs must stand on their own. `docs/README.md` (from `<templates>`
  `readme.md`) is a skill-agnostic entry point stating the doc structure and
  the operating discipline (work `phases.md` in order, check off tasks
  immediately, one commit per phase, log new ambiguities to `backlog.md`
  instead of assuming) in plain terms any AI agent or human can follow
  without knowing devpilot exists. Keep its document list in sync with
  `config.md`'s Active documents table whenever that list changes.
- **Core doc filenames are numbered for new projects only.** `init`/`onboard`
  generate `docs/core/` with a fixed numeric prefix per doc type (`00-config.md`
  … `09-backlog.md`, see Templates section) so the reading order matches
  `docs/README.md` without needing to open it. **Never rename an existing
  project's already-generated core docs to add this prefix** — that migration
  is explicitly out of scope; a project created before v0.2 keeps its
  unprefixed filenames indefinitely, and devpilot must locate/edit them the
  same way regardless (by content marker / base name, never a hardcoded path).
- **Phase file layout scales with project size, recorded in `config.md`'s
  Phase file layout row.** `single` (default): everything lives in one
  `phases.md` — Milestones table (if used) + Overview table + full `## Phase
  N` sections, per `core-phases.md`. `split`: `docs/core/phases/` becomes a
  directory — `00-overview.md` keeps the Milestones + Overview tables and
  status only, each phase's detail (goal, depends-on, task checkboxes) moves
  to its own `phase-{N}.md`. **Everywhere else in this file, "read/update
  `phases.md`" means whichever layout is active** — for `split`, that's
  `00-overview.md` for status/milestone changes and the matching `phase-{N}.md`
  for task-level changes. Propose switching `single` → `split` (init step 3,
  or when `feature`/`revise` would push phase count further while still on
  `single`) once the phase count is heading past roughly 12, or `phases.md` is
  approaching ~800 lines — ask once, explain the tradeoff in a sentence, record
  the choice, then perform the conversion yourself (lift each `## Phase N`
  section into its own file; never ask the user to do it manually). Add a
  Milestones table (`| Milestone | Phases | Status |` above the Overview
  table) independently of layout whenever a project has clearly separable
  modules/subsystems or a phase count that stops being scannable as a flat
  list — a project can have milestones without needing `split`.
- **Changelog logging is MANDATORY, never optional.** Every feature addition
  (`feature`), behavior change (`revise`), or bug fix (`fix`) — regardless of
  size — gets an entry in `changelog.md` classified by type
  (`feature`/`revision`/`bugfix`) and size (`small`/`medium`/`large`, your
  judgment based on scope of files/docs touched and complexity, not a fixed
  manday threshold). An entry is not "done" as a task until it is marked
  `done` in `changelog.md` **in the same commit** as the code change it
  describes — this is part of the definition of done, not a follow-up step
  that can be forgotten or deferred. If a change happens outside those three
  commands (an ad hoc edit during phase execution, say), still log it before
  considering the work finished.
- **Decision log entries are appended for consequential choices, not every
  choice.** Whenever `init`, `onboard`, `feature`, `revise`, or a phase
  in-flight settles a tradeoff that would surprise someone reading the docs
  cold — a tech/architecture choice made among real alternatives, a scope cut
  the user requested, a divergence from what the user originally described,
  or a structural call like switching phase file layout — append an entry to
  `config.md`'s **Decision log** (decision, rationale, alternatives
  considered). Routine implementation choices (variable names, which loop
  construct) never go here; if in doubt, ask "would the next person reading
  this be confused without it?" — only log if yes.
- **Security-sensitive work requires a mandatory review checkpoint — same
  severity as changelog logging, not optional diligence.** A task or phase
  touches auth/session handling, authorization/permission checks, payment or
  billing, personal/sensitive user data (PII), file upload, or
  cryptography/secret handling → treat it as security-sensitive. Before
  marking that task/phase done: run the security checklist explicitly as a
  visible step (Code quality standards, below) — don't fold it silently into
  "wrote the code." **If a dedicated security-review subagent/skill is
  available in this environment, use it; otherwise perform the checklist
  yourself** — either way this must actually happen, and say which one you
  did in the phase report. Record the outcome in the changelog entry's
  optional **Security review** field (`core-changelog.md`) when the work is
  logged. This gate cannot be silently skipped for triggered work, the same
  way a changelog entry cannot be silently skipped.
- **External requirement documents are optional input, not a replacement for
  the interview.** During `init`/`onboard`/`feature`/`revise`, ask once
  whether the user has existing documents describing the project or the
  change (ERD, SRS, FRS, technical specification, or any custom document).
  If yes, collect the file path(s), read them fully (the Read tool handles
  PDF, and any of the plain-text formats), and use their content to seed the
  relevant core docs (ERD → `database.md`; SRS/FRS → `requirements.md` +
  `features.md`; tech spec → `tech-stack.md` + `architecture.md`) — then run
  the interview only to close gaps the documents don't answer, never to
  re-ask what they already state. Record each ingested document in
  `config.md`'s **Source documents** section (name, type, date, which docs it
  fed) so provenance stays traceable if the docs and the source material
  later disagree. If no such document exists, proceed with the normal
  interview — this step never blocks or lengthens the flow when skipped.

## /devpilot init — new project

1. **Preflight:** if the target docs folder already exists with non-devpilot
   content, ask: merge into it, use a different folder name (recorded in
   config), or abort. If the folder already contains substantial source code,
   suggest `onboard` instead and confirm.
2. **Interview** (one question at a time). Before the first question, check
   for external requirement documents per Global Rules — if the user supplies
   any, ingest them first so the interview below only closes remaining gaps.
   Cover at minimum:
   1. Project goal, target users, and the problem being solved.
   2. Document language (Indonesian / English / other).
   3. Feature scope — probe until a concrete, enumerable module/feature list
      emerges. **Depth scales with signaled complexity**, still one question
      at a time: a simple single-purpose app needs one pass over the feature
      list, but a project with several interacting modules, multiple user
      roles with different permissions, multiple external integrations, or
      explicit compliance/regulatory needs warrants asking about each
      module's behavior and edge cases individually rather than one global
      pass — otherwise the gaps surface mid-implementation instead of now.
   4. Constraints: deadline, budget, required integrations, things it must not do.
   5. Tech stack preference — if the user has none, propose a stack that fits
      the project with reasoning, and confirm.
   6. Project type and team divisions — auto-propose divisions from project
      type (web app → Frontend, Backend, UI/UX, QA, DevOps; mobile app → those
      + Mobile; API/CLI-only → drop UI/UX), then let the user confirm or edit.
   7. UI/UX design direction — **only if the project has a user interface**.
      Run the design interview (below).

   Stop interviewing as soon as every applicable core doc can be written
   concretely — don't pad.
3. **Generate `docs/core/`** from `<templates>` (core-*.md) with numbered
   filenames (Templates section): `00-config`, `01-requirements`,
   `02-features` (per-division mandays for every feature, each row's
   Requirement(s) column citing the `FR-N` id(s) it implements — this is the
   traceability link back to `01-requirements`, keep it accurate as either
   doc changes), `03-tech-stack`,
   `04-architecture`, `05-design` (if applicable), `06-database` (if
   applicable), `07-api-contract` (if applicable), `08-phases` (ordered,
   dependency-aware plan; phases and tasks numbered `N` / `N.M`; propose
   milestones and/or the `split` layout per Global rules if the plan is
   large), `09-backlog` (empty scaffold), and `10-changelog` (empty scaffold
   — see Global rules).
   Also generate `docs/README.md` from `<templates>` `readme.md` (see Global
   rules), and `docs/formal/progress-dashboard.html` (empty-state data) so it
   exists from day one.
4. **Production hygiene** (section below).
5. **Git:** if the folder is not a git repository, offer `git init`.
6. Report the generated files, then ask whether to start executing Phase 1 now.

### Design interview (init step 2.7)

Runs only when the project has a user interface. Many users are not designers,
so **every question here is asked via AskUserQuestion with concrete options**,
each option described in plain language (what it looks/feels like, and what
kind of product it suits) — never assume design vocabulary, and never ask
open-ended "describe your design" questions. Still one question at a time.

1. **Visual style** — options tailored to the project type, e.g.:
   *Minimalist & clean* (lots of whitespace, few colors — content-focused
   apps); *Playful & colorful* (rounded shapes, bright accents — consumer or
   kids products); *Professional & corporate* (formal, trust-building —
   business/finance); *Bold & modern* (strong contrast, dark-friendly —
   tech/startup).
2. **Color direction** — first option "I have brand colors" (then collect
   them); otherwise propose 2–3 concrete palettes (named, with hex values)
   that match the chosen style, as options to pick from.
3. **Light/dark mode** — light only, dark only, or both (note that "both"
   costs extra effort; reflect it in mandays).
4. **Layout density** — *Spacious* (large elements, easy scanning — marketing
   pages, simple apps) vs *Compact* (information-dense — dashboards, admin
   tools).
5. **Target devices** — mobile-first, desktop-first, or fully responsive;
   propose a default based on the project type and target users.
6. **Reference apps/sites** — optional free text: products the user likes the
   look of and what they like about them. "None / just decide for me" is a
   valid answer.

Typography, component conventions (corner radius, shadows, icon set), and the
accessibility baseline are **not** interviewed one by one: propose a coherent
set of defaults derived from the answers above and ask for a single
confirmation. Record everything in `design.md`.

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
3. **Short interview:** check for external requirement documents per Global
   Rules first (e.g. a client's target-state spec for what onboarding should
   become, not just what exists). Then cover: document language; confirmation
   of detected stack/architecture; divisions (auto-propose + confirm); and
   anything the code cannot reveal (goals, target users).
4. **Generate `docs/core/`** (numbered filenames, see Templates section)
   describing **what already exists**, so future feature work does not
   conflict with current behavior. If the project has a UI, derive
   `design.md` from the existing frontend (UI framework/library, theme/palette
   from CSS or design tokens, layout patterns) instead of interviewing — only
   ask about design gaps the code cannot reveal. `phases.md` contains a single
   completed marker — `Phase 0 — Existing system (baseline)` — and no pending
   phases. `changelog.md` starts as an empty scaffold (existing history isn't
   backfilled). Also generate `docs/README.md` from `<templates>` `readme.md`
   and `docs/formal/progress-dashboard.html` (empty-state data) — see Global
   rules.
5. **Production hygiene** (section below).
6. Report the generated files and suggest next actions (`feature`, `docs`).

## /devpilot feature "<description>"

1. If no core docs exist → explain that docs are required first and route to
   `onboard` (or `init` if the folder is empty). Do not proceed.
2. Run a mini-interview **only if** the description is ambiguous.
3. **Impact analysis:** read the core docs and produce a table — affected doc →
   section → change summary. Examples: new table → `database.md` (+ migration
   note); new endpoints → `api-contract.md`; mandays delta → `features.md`
   (new/changed rows keep the Requirement(s) column accurate — add a new
   `FR-N` to `requirements.md` if the feature introduces a requirement that
   doesn't already have one); requirement changes → `requirements.md`; new
   screens or a changed visual direction → `design.md`; and the new phases
   that implementation would need.
4. Ask exactly **one** decision question: implement **now** or **later**?
5. - **Now** → open a `changelog.md` entry (type `feature`, size by judgment,
     status `in-progress`) → apply the updates to every affected core doc →
     append the new phase(s) with tasks to `phases.md` (still on `single`
     layout and this addition crosses the threshold in Global rules? propose
     `split` now, then append) → begin Phase execution. When the last of its
     phases completes, close the changelog entry (`done`, closing date,
     commit ref) in that same commit and regenerate
     `docs/formal/progress-dashboard.html` (see `/devpilot docs`).
   - **Later** → append an entry to `backlog.md`: date, description, the full
     impact analysis, status `deferred`. Touch **no other document** — the
     changelog entry is opened only when the work actually starts (step 6).
6. **Backlog promotion** (user asks to implement a backlog item, via `feature`
   or plain request): first re-validate the saved impact analysis against the
   **current** core docs — they may have changed since deferral. If the
   analysis is stale, regenerate it and show the user what changed before
   executing. Then proceed as "now" (including the changelog entry).

## /devpilot revise "<description>"

Same flow as `/devpilot feature`, for changes to a feature's **existing**
behavior rather than a net-new feature (e.g. "checkout should also accept
cash" on a system that already has checkout). Differences:

- Changelog entry type is `revision`, not `feature`.
- Impact analysis explicitly names the current behavior being replaced (read
  the relevant core doc section as it stands today, not just the delta), so
  the "before" is on record alongside the "after".
- Everything else — mini-interview if ambiguous, one now/later question,
  phase execution, closing the changelog entry in the completing commit,
  dashboard regeneration — is identical to `feature`.

## /devpilot fix "<description>"

Bug-fix flow. Unlike `feature`/`revise`, this has **no now/later choice** —
bugs are not deferred to the backlog by default (if the user explicitly wants
to defer one, route it through `feature`'s backlog path instead, noting it's
a fix).

1. If no core docs exist → same as `feature` step 1.
2. Mini-interview only if the description is ambiguous — in particular, ask
   for reproduction steps or the observed-vs-expected behavior if not given.
3. **Root cause + impact analysis:** identify the root cause (read the actual
   current source, per Global Rules — never assume), then the same affected
   doc → section → change table as `feature`. Most bugfixes touch no core doc
   beyond the changelog; note that plainly rather than padding the table.
4. Open a `changelog.md` entry (type `bugfix`, size by judgment, status
   `in-progress`, root cause recorded).
5. Fix it: add or extend a task under the current/relevant phase in
   `phases.md` if one fits, or a standalone entry if not — small fixes don't
   need a whole new phase.
6. Commit (`fix: …`), close the changelog entry (`done`, closing date, commit
   ref) **in that same commit**, and regenerate the progress dashboard.

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
  (least privilege); never leak sensitive data in error messages or logs. For
  auth/authorization/payment/PII/crypto work specifically, this checklist is
  the mandatory review checkpoint from Global rules, not just baseline style
  — run through it explicitly and record the outcome.
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

- Types: `prd`, `srs`, `gantt`, `dashboard`, or `all`.
- If no core docs exist → refuse, pointing to `init`/`onboard`.
- `prd`/`srs`/`gantt`: generate into `docs/formal/` from `<templates>`
  (formal-*.md), derived **entirely from the current `docs/core/` contents**
  — never from memory of past sessions. Every formal doc carries the
  generated-notice header from its template, filled with the generation
  date. Overwrite any previous version — formal docs are disposable outputs.
  `gantt` is a mermaid gantt chart with one section per division, planned
  dependency-aware with divisions working in parallel where dependencies
  allow.
- `dashboard`: **different regeneration rule from every other formal doc** —
  see the template's own header comment. If `docs/formal/progress-dashboard.html`
  doesn't exist yet, generate it in full from `<templates>`
  `formal-progress-dashboard.html`. If it already exists, **replace only the
  block between the `<!-- devpilot-data:start -->` / `<!-- devpilot-data:end
  -->` markers** with fresh JSON computed from `phases.md`, `backlog.md`, and
  `changelog.md` (schema in the template header) — never rewrite the
  surrounding HTML/CSS/JS shell. This also runs automatically whenever a
  phase completes or a changelog entry is opened/closed, not just on
  explicit invocation.
- `all`: every type above.

## /devpilot status

Show: per-phase progress (done/total tasks), the current or next pending task,
backlog item count, open changelog entries by type (features/revisions/
bugfixes still `open`/`in-progress`), the active-docs list from `config.md`,
and the most recent checkpoint commit (`git log --oneline` grep for phase
commits).

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
3. **Minimal CI scaffold** — deliberately narrow scope: a lint+test-on-push
   workflow, nothing more (no coverage threshold, no deploy step, no policy
   enforcement — that stays a non-goal). If the repo already has any CI
   config (`.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, etc.), skip
   this entirely — never touch existing CI setup. Otherwise, if the git
   remote points at `github.com` (the common case; skip silently for any
   other host or no remote — don't guess), propose generating
   `.github/workflows/ci.yml` (ask once): install dependencies, then run the
   project's **actual** existing lint/test commands as detected from its
   manifest (`package.json` scripts, `composer.json`, `pyproject.toml`,
   `go.mod` + `go test`, etc.) — never invent a command the project doesn't
   already have; if it has neither a lint nor a test script, say so and skip
   rather than generating an empty/fake workflow.

## Templates

`<templates>` contains one template per document. Core templates map to
numbered output filenames (new projects only — see Global rules):

| Template | Output filename |
|---|---|
| `core-config.md` | `00-config.md` |
| `core-requirements.md` | `01-requirements.md` |
| `core-features.md` | `02-features.md` |
| `core-tech-stack.md` | `03-tech-stack.md` |
| `core-architecture.md` | `04-architecture.md` |
| `core-design.md` | `05-design.md` (if applicable) |
| `core-database.md` | `06-database.md` (if applicable) |
| `core-api-contract.md` | `07-api-contract.md` (if applicable) |
| `core-phases.md` | `08-phases.md` |
| `core-backlog.md` | `09-backlog.md` |
| `core-changelog.md` | `10-changelog.md` |

- Formal: `formal-prd.md`, `formal-srs.md`, `formal-timeline-gantt.md`,
  `formal-progress-dashboard.html` (note its different regeneration rule,
  documented in its own header — `/devpilot docs` above).
- Meta: `readme.md` — generates `docs/README.md`, the skill-agnostic entry
  point (Global rules). Unlike core docs, this one is **never** skipped —
  it's not conditional on project type, it always applies.

Read the template before generating each document. Follow its structure, fill
every `{placeholder}`, translate headings into the configured document
language, and strip the instructional HTML comments from the generated output
(except the `<!-- devpilot-config -->` marker, which must be kept, and
`formal-progress-dashboard.html`'s comments, which stay — see its header).
