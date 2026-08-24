# devpilot

**Doc-driven project development for Claude Code.** devpilot turns your AI
assistant into a project copilot that interviews you, writes the planning
documents, executes the work in resumable phases — and never loses the plot
between sessions, because every decision lives in markdown, not in chat
history.

## Why

AI agents drift: they forget decisions from three sessions ago, re-architect
things mid-flight, and die mid-task when the context window runs out. The fix
is old-fashioned — write things down. devpilot enforces that discipline:

- **`docs/core/`** — the single source of truth (requirements, features +
  mandays, stack, architecture, design, database, API contract, phases,
  backlog, changelog). Every change updates these files first.
- **`docs/formal/`** — presentation documents (PRD, SRS, gantt timeline,
  progress dashboard), generated on demand from core. They can never drift
  out of date because they're never hand-maintained — regenerate anytime.
- **Interruption-proof execution** — work is planned as numbered phases with
  task-level checkboxes, checked off in real time and git-committed per phase.
  Hit a token limit mid-task? A fresh session with `/devpilot resume` picks up
  exactly where the last one died.
- **Nothing ships undocumented** — every feature, revision, and bug fix is
  logged in `changelog.md` (typed, sized, dated) and isn't "done" until the
  entry is marked done in the same commit as the code.
- **Scales to large projects** — phases can group under milestones, and
  `phases.md` splits itself into one file per phase (devpilot does the move,
  not you) once a plan grows past ~12 phases or ~800 lines.
- **Traceable and audited** — every feature row in `features.md` cites the
  `FR-N` requirement it implements; consequential tradeoffs get a decision-log
  entry (lite ADR) in `config.md`; and auth/payment/PII/crypto work runs
  through a mandatory security-review checkpoint before it can be marked done.

## Install

Requires [Claude Code](https://claude.com/claude-code). No other dependencies —
devpilot is a pure markdown skill.

```bash
git clone https://github.com/Taufiq26/devpilot
cp -r devpilot/skills/devpilot ~/.claude/skills/devpilot
```

Or per-project instead of global:

```bash
cp -r devpilot/skills/devpilot YOUR_PROJECT/.claude/skills/devpilot
```

Then type `/devpilot` in Claude Code.

## Usage

| Command | What it does |
|---|---|
| `/devpilot` | Auto-detects context: offers `init`, `onboard`, `resume`, or `status` |
| `/devpilot init` | New project: guided interview (or ingest an existing ERD/SRS/FRS/tech spec) → full `docs/core/` set → phased plan |
| `/devpilot onboard` | Existing project: analyzes the codebase → documents what exists |
| `/devpilot feature "…"` | Impact analysis → asks *implement now or later?* → updates docs & phases, or files it to the backlog |
| `/devpilot revise "…"` | Same flow as `feature`, for changing an existing feature's behavior |
| `/devpilot fix "…"` | Bug fix: root cause + impact analysis → implements immediately, logged to `changelog.md` |
| `/devpilot resume` | Continues interrupted execution from the exact task where it stopped |
| `/devpilot docs prd\|srs\|gantt\|dashboard\|all` | Generates formal documents into `docs/formal/` from current core docs |
| `/devpilot status` | Phase/task progress, backlog count, open changelog entries, last checkpoint |

### A typical flow

```
/devpilot init          # interview → docs/core/ generated → phase plan
                        # → phases execute automatically, committing per phase
[token limit hits mid-phase 3]
/devpilot resume        # new session reads phases.md + git status,
                        # finishes task 3.3, continues to phase 4
/devpilot feature "customer loyalty points"
                        # → impact analysis: database.md, api-contract.md,
                        #   features.md (+8 mandays), 2 new phases
                        # → "now or later?" → later → backlog.md only
/devpilot fix "checkout crashes on empty cart"
                        # → root cause + impact → fixes it → changelog.md
                        #   logs a bugfix entry, marked done in the same commit
/devpilot docs all      # fresh PRD, SRS, mermaid gantt, and progress
                        # dashboard (HTML) in docs/formal/
```

## What gets generated

```
docs/
├── core/                       # single source of truth — updated on every change
│   ├── 00-config.md            # language, divisions, active docs, source documents
│   ├── 01-requirements.md      # FR/NFR, users, out-of-scope
│   ├── 02-features.md          # feature list + per-division mandays
│   ├── 03-tech-stack.md        # choices + reasoning
│   ├── 04-architecture.md      # mermaid diagram, environments, server specs
│   ├── 05-design.md            # visual style, palette, layout (if there's a UI)
│   ├── 06-database.md          # ERD + schema + migration log
│   ├── 07-api-contract.md      # endpoint contracts
│   ├── 08-phases.md            # numbered phases, task checkboxes, status
│   ├── 09-backlog.md           # deferred features with saved impact analyses
│   └── 10-changelog.md         # mandatory log: every feature/revision/bugfix,
│                                #   typed and sized, marked done when actually done
└── formal/                     # generated on demand — never hand-edited
    ├── PRD.md
    ├── SRS.md
    ├── timeline-gantt.md       # mermaid gantt per division
    └── progress-dashboard.html # standalone PM/client report — open in any
                                 #   browser, no server; filterable by type/size/status
```

The numeric prefixes make the reading order visible directly in a file
browser, matching the order above — no need to open this README first to
know what to read next. Projects created before this convention keep their
original (unprefixed) filenames; devpilot locates core docs by content, not
by a hardcoded path, so both styles work.

See [examples/warung-pos](examples/warung-pos) for real sample output,
including a mid-execution `phases.md` and a generated gantt.

## Notes

- **Document language** is asked once per project (e.g. English or Indonesian)
  and applies to everything generated for that project.
- **Existing requirement documents are optional input:** `init`/`onboard`/
  `feature`/`revise` ask once whether you already have an ERD, SRS, FRS,
  technical spec, or any custom document — if so, devpilot reads it and seeds
  the relevant core docs from it, then only interviews you on what the
  document doesn't answer.
- **Production hygiene:** devpilot adds `docs/`, `CLAUDE.md`, `AGENTS.md` and
  similar AI files to your stack's build-exclusion files (`.dockerignore`,
  `.vercelignore`, …) so they stay in git but out of production builds. If
  the project has a GitHub remote and no CI yet, it can also scaffold a
  minimal `.github/workflows/ci.yml` that runs your project's own existing
  lint/test commands — nothing invented, no coverage policy imposed.
- **[graphify](https://github.com/Graphify-Labs/graphify) integration is
  optional:** if installed, `onboard` uses it to analyze large codebases
  faster; everything works without it.
- Mandays and timelines are planning estimates, not commitments.

## License

[MIT](LICENSE)
