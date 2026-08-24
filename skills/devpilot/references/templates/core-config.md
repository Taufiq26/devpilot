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
| devpilot version | 0.2.0 |
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
