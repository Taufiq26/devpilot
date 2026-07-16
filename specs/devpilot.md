# devpilot — Doc-Driven Project Development Skill

## Objective

devpilot is a Claude Code skill (publishable on GitHub, installable by others like
graphify or impeccable) that makes AI-assisted project development consistent and
interruption-proof by anchoring all work to a structured set of markdown documents.
It guides the user from a blank folder (or an existing codebase) through a guided
interview, generates a layered documentation set (`docs/core/` as the single source
of truth, `docs/formal/` generated on demand), plans work as sequential executable
phases with task-level progress tracking, and handles mid-project feature additions
through impact analysis with an explicit "implement now vs. backlog" decision.
The target user is a developer (initially the author, then the public) who wants an
AI agent to build/extend projects without losing context between sessions.

## Requirements

### Packaging & distribution

1. devpilot is a **pure markdown skill** for Claude Code: a repo containing a skill
   folder (`SKILL.md` + `references/` templates), no binaries, no Python/npm
   package, no installer CLI. Installation = clone/copy the skill folder into
   `~/.claude/skills/devpilot/` (global) or `.claude/skills/devpilot/` (project).
2. The GitHub repo must contain: the skill folder, a `README.md` (English) with
   install + usage instructions, a `LICENSE` (MIT), and at least one worked example
   showing generated output.
3. The repo layout must not prevent a future upgrade to a Claude Code
   plugin/marketplace distribution (keep skill self-contained in one folder).

### Command surface

4. One command, `/devpilot`, with subcommands:
   - `/devpilot` (bare) — auto-detect context: no `docs/core/` → offer `init`
     (empty/near-empty folder) or `onboard` (existing code); unfinished phases →
     offer `resume`; otherwise show `status` and available actions.
   - `/devpilot init` — new-project guided interview → full doc generation → phase plan.
   - `/devpilot onboard` — existing-project analysis → doc generation reflecting
     current code structure and behavior.
   - `/devpilot feature "<description>"` — feature-addition flow (impact analysis →
     now/later).
   - `/devpilot resume` — continue interrupted phase execution.
   - `/devpilot docs <type>` — generate a formal document (`prd`, `srs`, `gantt`,
     or `all`) from core docs.
   - `/devpilot status` — show phase/task progress summary.

### Documentation structure (core + formal)

5. All generated docs live in one folder, default `docs/` (name customizable at
   init/onboard, stored in config), with two tiers:
   - `docs/core/` — **the single source of truth**, updated on every change:
     - `config.md` — project meta: doc language, docs folder name, divisions,
       project type, stack summary.
     - `requirements.md` — condensed product + software requirements.
     - `features.md` — feature/module list, each with per-division mandays estimates.
     - `tech-stack.md` — chosen stack with reasoning.
     - `architecture.md` — ideal server/deployment architecture.
     - `database.md` — ERD (mermaid) + schema; changes carry migration notes.
     - `api-contract.md` — endpoint contracts (omit for projects with no API).
     - `phases.md` — sequential execution phases with task-level checkboxes and
       per-phase status.
     - `backlog.md` — deferred features, each with its saved impact analysis.
   - `docs/formal/` — **generated on demand only**, never hand-maintained:
     `PRD.md`, `SRS.md`, `timeline-gantt.md` (mermaid gantt per division). Each
     regeneration derives fully from current `docs/core/`, so formal docs can
     never drift from core.
6. Formal docs must carry a header noting they are generated from `docs/core/` with
   a generation date.

### Init flow (new project)

7. `init` runs a guided interview, **one question at a time**, covering at minimum:
   project goal/users, document language (Indonesian/English — asked once, stored
   in `config.md`, all core+formal docs follow it), feature scope, constraints,
   tech-stack preference (propose if user has none), and team divisions.
8. Divisions are **auto-proposed from project type** (e.g. web app → Frontend,
   Backend, UI/UX, QA, DevOps; mobile → + Mobile) and confirmed/edited by the user.
9. After the interview, `init` generates all applicable `docs/core/` files,
   including `features.md` with per-division mandays and `phases.md` with an
   ordered, dependency-aware phase plan. Timeline/gantt is generated into
   `docs/formal/` on demand (requirement 5), planned as if each division works in
   parallel where dependencies allow.

### Onboard flow (existing project)

10. `onboard` analyzes the existing codebase (structure, stack, database schema,
    API surface, observable behavior) and generates the same `docs/core/` set
    describing **what already exists**, so future feature work does not conflict
    with current behavior. `phases.md` starts empty (no pending phases) plus a
    completed "Phase 0: existing system" marker.
11. If `graphify-out/graph.json` exists or the `graphify` CLI is available, onboard
    should use it to accelerate/deepen code analysis; graphify must remain an
    **optional** enhancement — onboard must work without it.

### Feature-addition flow

12. `feature` runs: (a) a mini-interview if the description is ambiguous, (b) an
    **impact analysis** listing every core doc section affected (e.g. new table →
    `database.md`, new endpoints → `api-contract.md`, mandays delta →
    `features.md`), then (c) asks exactly one decision question: **implement now
    or later?**
    - **Now** → update all affected core docs, append new phases to `phases.md`,
      then begin phase execution.
    - **Later** → append the feature + its full impact analysis to `backlog.md`;
      touch no other doc.
13. Promoting a backlog item ("kerjakan backlog X" / via `feature`) must first
    **re-validate the saved impact analysis** against current core docs (they may
    have changed since deferral) before executing.

### Phase execution & resume

14. Execution proceeds phase by phase, **auto-continuing without approval pauses**:
    finish phase → update `phases.md` → `git commit` (checkpoint) → brief report →
    next phase. The user can interrupt at any time.
15. Task checkboxes in `phases.md` are updated **immediately as each task
    completes** (not batched at phase end), so on-disk state always reflects true
    progress even after a hard interruption.
16. `resume` recovers by: reading `phases.md` for the first unchecked task →
    checking `git status`/`git diff` for uncommitted partial work → inspecting any
    half-finished files → completing (or cleanly redoing) only the interrupted
    task, then continuing normally. Prior completed phases are never redone.
17. Commit messages follow conventional-commit format and reference the phase
    (e.g. `feat: phase 3 — auth module`).
18. All phase implementation work follows **senior-developer code quality
    standards**, stated explicitly in the skill's phase-execution instructions:
    language/framework best practices and existing project conventions,
    security-aware coding (input validation at boundaries, parameterized
    queries, no hardcoded secrets, output escaping, least-privilege
    authorization, no sensitive data in errors/logs), explicit error handling,
    and readable maintainable structure (naming, small functions, early
    returns, no magic numbers).

### Production hygiene

19. During init/onboard, devpilot adds the AI/doc paths (`docs/` [or custom name],
    `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md`, `.claude/`, and similar agent
    files) to the **build/deploy exclusion files** appropriate to the detected
    stack — `.dockerignore`, `.vercelignore`, `.gcloudignore`, `.npmignore`, etc.
    These files stay committed on all git branches (including `main`); exclusion
    happens at build/deploy level only. No branch-conditional `.gitignore`
    mechanics.

## Constraints

- **Markdown-only skill**: no runtime dependencies, no network calls, no installer.
  Everything the skill does is instruction-driven through Claude Code's normal
  tools (Read/Write/Edit/Bash/git).
- Claude Code is the only target platform for v1. Multi-platform (Cursor, Codex,
  Gemini CLI) and marketplace-plugin packaging are explicit non-goals for v1.
- Generated project docs follow the per-project language choice; the **repo's own
  README and skill instructions are in English** for international adoption.
- graphify integration is optional-only (requirement 11); the skill must never
  fail or degrade confusingly when graphify is absent.
- Mandays/timeline figures are presented as **estimates for planning/presentation**
  ("as if executed by a team per division"), not commitments; templates must label
  them as such.
- Phase execution requires a git repository (for checkpoints); see edge cases for
  the non-git flow.
- Non-goals: no time tracking, no external PM-tool integration (Jira etc.), no
  multi-user collaboration features, no enforcement of the 80%-coverage/TDD policy
  on target projects (the skill may recommend tests as phase tasks, but testing
  policy belongs to the target project, not devpilot).

## Edge Cases

1. **`docs/` already exists with other content** (onboard/init): detect, then ask —
   merge into it, use a different folder name (stored in config), or abort.
2. **Existing `CLAUDE.md`/`AGENTS.md`**: never overwrite; append a clearly marked
   devpilot section instead, preserving user content.
3. **Not a git repo** when phase execution starts: offer `git init`; if declined,
   execute without commit checkpoints and warn that resume granularity falls back
   to `phases.md` checkboxes only.
4. **Interrupted mid-task** (token limit/crash): handled by requirement 16 — at
   most one task's partial work is affected, recoverable from the working tree.
5. **`/devpilot resume` with nothing pending**: report all phases complete and
   suggest `status` / `feature`.
6. **`/devpilot feature` before init/onboard**: explain docs are required first and
   route to `onboard` (or `init` if the folder is empty).
7. **User manually edited core docs**: treat edits as authoritative (core is the
   source of truth); formal regeneration and future planning must respect them.
   If a manual edit contradicts `phases.md` state, surface the conflict and ask.
8. **Stale backlog impact analysis**: covered by requirement 13 — re-validate
   before executing; if the analysis no longer holds, regenerate it and show the
   diff to the user.
9. **Project with no API/database** (e.g. static site, CLI tool): skip
   `api-contract.md`/`database.md` rather than generating empty shells; `config.md`
   records which core docs are active.
10. **`/devpilot docs <type>` before core docs exist**: refuse with a pointer to
    `init`/`onboard`.
11. **Very large existing codebase on onboard**: analyze incrementally
    (prioritize entry points, schema, routes) and state explicitly in generated
    docs which areas were sampled vs. fully analyzed; recommend graphify when the
    codebase exceeds comfortable direct analysis.

## Definition of Done

- [ ] Repo contains `skills/devpilot/SKILL.md` + `references/` templates for every
      core doc (`config`, `requirements`, `features`, `tech-stack`, `architecture`,
      `database`, `api-contract`, `phases`, `backlog`) and every formal doc
      (`PRD`, `SRS`, `timeline-gantt`).
- [ ] Copying the skill folder into `~/.claude/skills/` makes `/devpilot` invocable
      in Claude Code with all seven behaviors from requirement 4.
- [ ] **Init test**: running `/devpilot init` in an empty folder conducts a
      one-question-at-a-time interview (incl. language + division confirmation) and
      produces a complete `docs/core/` set with per-division mandays in
      `features.md` and an ordered phase plan in `phases.md`.
- [ ] **Onboard test**: running `/devpilot onboard` on a real existing repo
      produces core docs that accurately describe its current stack, schema, and
      behavior, with an empty pending-phase list; works with graphify absent.
- [ ] **Feature test**: `/devpilot feature` on the onboarded project yields an
      impact analysis naming affected docs, asks now/later; "later" writes only
      `backlog.md`; "now" updates affected core docs and appends phases.
- [ ] **Interrupt/resume test**: killing a session mid-task, then `/devpilot
      resume` in a fresh session, continues from the interrupted task without
      redoing completed phases (verified via `phases.md` + `git log`).
- [ ] **Formal docs test**: `/devpilot docs all` generates PRD, SRS, and a mermaid
      gantt into `docs/formal/`, each consistent with current core docs and marked
      with a generation header; regenerating after a core change reflects the change.
- [ ] Build-exclusion files (`.dockerignore` etc.) matching the detected stack are
      created/updated during init/onboard per requirement 19.
- [ ] README (English) documents installation, all subcommands, the core/formal
      concept, and includes a worked example; LICENSE present.
- [ ] Edge cases 1–3, 5, 6, and 10 verified by manual test on a scratch project.
