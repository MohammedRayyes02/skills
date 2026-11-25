# Backend Engineering Reference

Read this when working on APIs, data access, authentication, or server-side business logic. Framework conventions (Laravel, Django, Rails, Express, Spring, etc.) take precedence over the generic shapes below; use the framework's own validation, ORM, policy, and middleware features.

## Contents
- Structure and responsibilities
- Input validation
- Authentication and authorization
- Data access and transactions
- Errors and responses
- API design
- Sensitive data
- Migrations and schema changes
- Checklist

## Structure and responsibilities

- Keep transport (routing, request parsing, response formatting), business rules, and persistence in separate places, using the layering the project already has. Do not introduce a new layer (service, repository, DTO) unless the project uses it or the logic genuinely needs it.
- Business rules belong where they can be tested without HTTP or a database, not inside controllers, serializers, or SQL strings.
- Controllers/handlers stay thin: validate, authorize, call the domain logic, shape the response.

## Input validation

- Treat every external input as untrusted: request body, query, headers, path params, file uploads, webhooks, env values, third-party API responses.
- Validate shape, type, range, length, and allowed values at the boundary, using the framework's validation facility. Reject by default; allow-list known fields (guard against mass assignment).
- Validate uploads by type, size, and content, not just filename extension; store them outside the web root or with safe handling.
- Never build SQL, shell commands, file paths, or HTML by concatenating input. Use parameterized queries, argument arrays, path normalization, and output encoding.

## Authentication and authorization

- Authentication answers "who are you"; authorization answers "may you do this to this object." Do both, every time.
- Check authorization on the server for every request that reads or changes a resource. Hiding a button in the UI is not a control.
- Prevent insecure direct object references: when loading a resource by id, scope the query to what the current user may access (or check ownership/policy) before returning or mutating it. Example: fetch `invoice WHERE id = ? AND account_id = current_account`, not `invoice WHERE id = ?`.
- Use the framework's established auth mechanism (middleware, policies, guards). Do not roll your own crypto, session handling, or password hashing.
- Apply least privilege and deny by default; new endpoints should be protected unless intentionally public.

## Data access and transactions

- Avoid N+1 queries: eager-load or join relations used in a loop. Select only needed columns for large reads. Paginate unbounded lists.
- Use transactions when several writes must succeed or fail together (money movement, creating a record plus its children, state transitions). Keep them short and free of network calls.
- Guard against races on read-modify-write (unique constraints, row locks, optimistic version checks) where correctness depends on it.
- Enforce integrity in the database too (foreign keys, unique and not-null constraints), not only in application code.
- Add indexes for columns used in frequent filters, joins, and ordering, after checking real query patterns.

## Errors and responses

- Handle errors consistently through the project's central mechanism (exception handler, error middleware), not ad hoc per endpoint.
- Return correct status codes (400/422 validation, 401 unauthenticated, 403 forbidden, 404 missing, 409 conflict, 5xx server fault) and a consistent error body.
- Log enough context to debug (request id, relevant ids) without logging secrets or personal data. Do not leak stack traces, SQL, or internals to clients.
- Do not swallow exceptions. Translate them at boundaries, and fail loudly on genuinely unexpected states.

## API design

- Predictable contracts: consistent naming, pluralization, pagination shape, error shape, and date/time format (ISO 8601, UTC).
- Make safe methods safe (`GET` has no side effects) and make retried writes idempotent where clients may retry (idempotency keys, upsert semantics).
- Do not break existing consumers: add fields rather than change meaning, version or deprecate deliberately, and update all callers when you must change a contract.

## Sensitive data

- No secrets, keys, or tokens in source, logs, or error messages; read them from configuration.
- Hash passwords with the framework's recommended algorithm. Encrypt sensitive fields where required. Hide sensitive attributes from serialization by default.
- Collect and return only the data the feature needs.

## Migrations and schema changes

- Make migrations reversible when feasible and safe to run on existing data (nullable-then-backfill-then-constrain for large tables).
- Never edit an already-applied migration; add a new one.
- Call out destructive changes (drops, type narrowing) and get approval before applying.

## Checklist

- [ ] Inputs validated at the boundary; fields allow-listed
- [ ] Authorization checked per object, scoped queries
- [ ] No N+1; lists paginated
- [ ] Multi-write operations in a transaction
- [ ] Errors handled consistently, nothing sensitive leaked
- [ ] Existing API contracts preserved
- [ ] Project's tests, lint, and static analysis run
