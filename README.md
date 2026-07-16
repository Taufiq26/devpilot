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
  mandays, stack, architecture, database, API contract, phases, backlog).
  Every change updates these files first.
- **`docs/formal/`** — presentation documents (PRD, SRS, gantt timeline),
  generated on demand from core. They can never drift out of date because
  they're never hand-maintained — regenerate anytime.
- **Interruption-proof execution** — work is planned as numbered phases with
  task-level checkboxes, checked off in real time and git-committed per phase.
  Hit a token limit mid-task? A fresh session with `/devpilot resume` picks up
  exactly where the last one died.

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
| `/devpilot init` | New project: guided interview → full `docs/core/` set → phased plan |
| `/devpilot onboard` | Existing project: analyzes the codebase → documents what exists |
| `/devpilot feature "…"` | Impact analysis → asks *implement now or later?* → updates docs & phases, or files it to the backlog |
| `/devpilot resume` | Continues interrupted execution from the exact task where it stopped |
| `/devpilot docs prd\|srs\|gantt\|all` | Generates formal documents into `docs/formal/` from current core docs |
| `/devpilot status` | Phase/task progress, backlog count, last checkpoint |

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
/devpilot docs all      # fresh PRD, SRS, mermaid gantt in docs/formal/
```

## What gets generated

```
docs/
├── core/                  # single source of truth — updated on every change
│   ├── config.md          # language, divisions, active docs
│   ├── requirements.md    # FR/NFR, users, out-of-scope
│   ├── features.md        # feature list + per-division mandays
│   ├── tech-stack.md      # choices + reasoning
│   ├── architecture.md    # mermaid diagram, environments, server specs
│   ├── database.md        # ERD + schema + migration log
│   ├── api-contract.md    # endpoint contracts
│   ├── phases.md          # numbered phases, task checkboxes, status
│   └── backlog.md         # deferred features with saved impact analyses
└── formal/                # generated on demand — never hand-edited
    ├── PRD.md
    ├── SRS.md
    └── timeline-gantt.md  # mermaid gantt per division
```

See [examples/warung-pos](examples/warung-pos) for real sample output,
including a mid-execution `phases.md` and a generated gantt.

## Notes

- **Document language** is asked once per project (e.g. English or Indonesian)
  and applies to everything generated for that project.
- **Production hygiene:** devpilot adds `docs/`, `CLAUDE.md`, `AGENTS.md` and
  similar AI files to your stack's build-exclusion files (`.dockerignore`,
  `.vercelignore`, …) so they stay in git but out of production builds.
- **[graphify](https://github.com/Graphify-Labs/graphify) integration is
  optional:** if installed, `onboard` uses it to analyze large codebases
  faster; everything works without it.
- Mandays and timelines are planning estimates, not commitments.

## License

[MIT](LICENSE)
