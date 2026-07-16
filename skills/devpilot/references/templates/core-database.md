# Database Design

## ERD

```mermaid
erDiagram
    USERS ||--o{ ORDERS : places
    USERS {
        bigint id PK
        varchar email
    }
    ORDERS {
        bigint id PK
        bigint user_id FK
    }
```

## Tables

### {table_name}

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | {bigint} | PK | |
| {…} | {…} | {…} | {…} |

## Migration log

<!-- Append-only. Every schema change gets a row, linked to the feature/phase that caused it. -->
| Date | Change | Reason / feature | Phase |
|---|---|---|---|
| {YYYY-MM-DD} | {initial schema} | {init} | 1 |
