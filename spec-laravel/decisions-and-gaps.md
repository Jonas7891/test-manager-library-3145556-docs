# Decisions on gaps and ambiguities in the spec

Each row is a point where the spec is silent, contradicts itself or allows two readings. The decision **does not widen the
scope**: it only picks one reading and writes it down. Decisions that affect the architecture have their own ADR. The full list of
faults in the base commit is in [`base-spec-audit.md`](base-spec-audit.md).

| # | Gap / ambiguity | Where | Decision |
|---|---|---|---|
| H-01 | ADR-002 and D-04 mention a "version token per book", but `data-model.md` §3, which has priority, has no `version` column | ADR-002 vs data-model §3 | `available_copies` acts as the token (compare-and-set). No column is added. [ADR-005](adr/adr-005-concurrency-token-without-version-column.md) |
| H-02 | T-01 and CA-01.3 ask for `GET /health`, but the contract sets the base path `/api/v1` | tasks / contract §1 | Both `/api/v1/health` and `/health` are served, with the same response |
| H-03 | Contract §3.4 mentions `GET /books/{id}/cover` in the text but not in the endpoint table | contract §3.4 | Implemented for authenticated users; returns the binary or 404. [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| H-04 | `GET /loans/overdue` clashes with `GET /loans/{id}` | contract §3.6 | `overdue` is declared first and `{id}` only accepts a UUID |
| H-05 | T-21 asks for 3 overdue loans created **through the API**, but the API stamps each loan with its own clock | T-21 vs HU-06 | The system clock accepts a `CLOCK_OFFSET_DAYS` shift (0 by default). [ADR-006](adr/adr-006-shiftable-clock-for-the-seeder.md) |
| H-06 | CA-06.4 asks for "422 with the current count" but does not say where the count goes | spec / contract | In `message` and in `details: [{field: "member_id", issue: "active_loans=N"}]` |
| H-07 | Laravel answers 401, 404 and 422 with its own format | RNF-4 | A single error renderer turns every exception into the `{code, message, details}` body |
| H-08 | Who creates the `library` schema? | ADR-001 | The first migration (`CREATE SCHEMA IF NOT EXISTS library`). `infra` only creates the engine, the user and the volume |
| H-09 | `PUT /books/{id}` does not include the ISBN | contract §3.4 | The ISBN cannot be changed once the book exists |
| H-10 | The `status` filter of `GET /loans` uses a status that is not stored | HU-08 | Computed in SQL with `EXISTS` over `loan_item` (`returned_at IS NULL`, `due_at < now`) |
| H-11 | The report's `from` and `to` are dates, "both inclusive", but instants are UTC | contract §3.7 | `loaned_at >= from 00:00Z` and `loaned_at < (to + 1 day) 00:00Z` |
| H-12 | A soft-deleted book must still show in past loans | ADR-003 | Lines are read with their frozen values; they are never joined to the live catalogue |
| H-13 | `GET /loans/overdue` is a list: is it paginated? | contract §3.6 | It returns `{items, page, size, total}` like every other list |
| H-14 | `GET /members` accepts `q`, but does not say which fields it searches | CA-05.2 | Case-insensitive partial match on `full_name` or `document_number` |
| H-15 | Can copies be reduced below the number on loan? | CA-03.3 | No: the difference is applied to the available copies; if they fall below 0 → 422 `VALIDATION_ERROR` |
| H-16 | Contract §1 requires 401 with no token or an expired token, but catalogue §2 has no code for that case | contract §1 / §2 | `INVALID_CREDENTIALS`, the only 401 code in the catalogue, is used with the message "Se requiere un token válido y vigente.". No codes are invented. [ADR-008](adr/adr-008-reuse-invalid-credentials-for-missing-token.md) |
| H-17 | `PUT /books/{id}/cover` is `multipart/form-data`, but PHP does not parse multipart on a real `PUT` | contract §3.4 | The route is kept; the client sends `POST` with `_method=PUT` (Laravel method spoofing). [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| H-18 | `artisan serve` filters the environment variables of its processes, so the API ended up using SQLite | RNF-5 | The container serves with PHP's built-in server, which inherits the environment. `pgsql` is also the default connection |
