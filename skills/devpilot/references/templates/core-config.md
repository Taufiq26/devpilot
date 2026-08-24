<!-- devpilot-config -->
<!-- Template: keep the marker comment above in generated output. -->
# Project Configuration

| Key | Value |
|---|---|
| Project name | {name} |
| Project type | {web app / mobile app / API service / CLI tool / …} |
| Document language | {Indonesian / English / …} |
| Docs folder | {docs} |
| Created | {YYYY-MM-DD} |
| devpilot version | 0.2.1 |
| Core doc filenames | numbered ({00-config.md} …) |
| Phase file layout | {single / split — see SKILL.md Global rules} |

## Divisions

<!-- The confirmed team divisions used for estimates and the gantt. -->
- {Frontend}
- {Backend}
- {UI/UX}
- {QA}
- {DevOps}

## Stack summary

{One line, e.g. "Laravel 11 + Vue 3 + PostgreSQL 16, deployed on Docker/GCP"}

## Active documents

<!-- Which core docs exist for this project; state why any were skipped. -->
| Document | Active | Note |
|---|---|---|
| requirements.md | yes | |
| features.md | yes | |
| tech-stack.md | yes | |
| architecture.md | yes | |
| design.md | {yes/no} | {skipped: project has no user interface} |
| database.md | {yes/no} | {skipped: project has no database} |
| api-contract.md | {yes/no} | {skipped: project has no API} |
| phases.md | yes | |
| backlog.md | yes | |
| changelog.md | yes | |

## Source documents

<!-- Only include this section if the user supplied external requirement
     documents (ERD/SRS/FRS/tech spec/custom) during init/onboard/feature.
     Omit the section entirely otherwise — don't leave it as "none". -->
| Document | Type | Ingested | Used for |
|---|---|---|---|
| {client-srs.pdf} | {SRS} | {YYYY-MM-DD} | {requirements.md, features.md} |

## Decision log

<!-- Lite ADR (Architecture Decision Record). One entry per consequential
     decision — tech/architecture tradeoffs, scope cuts, a divergence from
     what the user originally asked for, or a structural choice like the
     phase file layout switch. NOT every small choice — if it wouldn't
     surprise someone reading the docs cold, it doesn't need an entry. See
     SKILL.md Global rules for when to append one. -->
_No decisions logged yet._

<!-- Entry format:

### {YYYY-MM-DD} — {short decision title}

- **Decision:** {what was decided}
- **Rationale:** {why, in the context at the time}
- **Alternatives considered:** {what else was on the table, and why not}
-->
