# API Contract

## Conventions

- Base URL: `{/api/v1}`
- Auth: {e.g. Bearer JWT in Authorization header}
- Response envelope:

```json
{ "success": true, "data": {}, "error": null, "meta": {} }
```

## Endpoints

### {POST /auth/login}

- **Auth:** {none}
- **Request:**

```json
{ "email": "user@example.com", "password": "…" }
```

- **Response 200:**

```json
{ "success": true, "data": { "token": "…" }, "error": null }
```

- **Errors:** {401 invalid credentials, 422 validation}
