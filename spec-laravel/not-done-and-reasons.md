# What could not be done exactly as the spec says, and why

Applying the spec (constitution, ADRs, plan, tasks and contract) brought up points that **could not be met to the letter**. In
each case the document contradicts itself, the challenge stack does not allow it, or the evidence depends on something that does
not exist in this delivery. Each case states **what the spec says**, **what prevented it** and **what was done instead**. None of
them was hidden by editing the original spec: `spec-python/` and `spec-.net/` are untouched. The full audit of the base commit is
in [`base-spec-audit.md`](base-spec-audit.md).

## 1. Errors or contradictions in the spec itself

| # | What the spec says | Problem found | What was done |
|---|---|---|---|
| E-1 | ADR-002 and D-04: optimistic control with a **"version token" per book** | `data-model.md` §3, which says it wins over any other document, **has no `version` column**. Following the ADR meant changing the model with no requirement asking for it | **Not done as the ADR says.** The token is the value read from `available_copies` (compare-and-set). See [ADR-005](adr/adr-005-concurrency-token-without-version-column.md). Tested in `ConcurrencyTest` |
| E-2 | ADR-002: on conflict, 409 "**or retry once** if the copy is still available" | An open alternative ("or") that the contract does not reflect; retrying would hide from the librarian that someone else changed the book | **The retry was not implemented.** It always answers 409 `BOOK_NOT_AVAILABLE`, the ADR's first option |
| E-3 | T-21: the seeder creates **3 overdue loans using the API** | Impossible as written: the server clock sets `loaned_at` (HU-06) and the contract accepts no dates. A loan created today is not overdue for 14 days | Shiftable clock for the demo only (`CLOCK_OFFSET_DAYS`, 0 by default). See [ADR-006](adr/adr-006-shiftable-clock-for-the-seeder.md) |
| E-4 | Contract §1: 401 with no token or an expired token | Code catalogue §2 **has no code** for that case (`INVALID_CREDENTIALS` is for credentials) | `INVALID_CREDENTIALS` is reused with another message instead of inventing a code outside the contract. See [ADR-008](adr/adr-008-reuse-invalid-credentials-for-missing-token.md) |
| E-5 | T-01 and CA-01.3: `GET /health`; contract §1: base `/api/v1` | The two paths do not match | Both are served (H-02) |
| E-6 | Contract §3.4: `GET /books/{id}/cover` appears in the text but not in the endpoint table | Endpoint with no formal role | Implemented for any authenticated user. See [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| E-7 | Contract §3.6: `/loans/overdue` and `/loans/{id}` | The `overdue` path could be read as an id | `overdue` is declared first and `{id}` only accepts a UUID (H-04) |

## 2. What the challenge stack cannot do the same way

| # | What the spec says | Problem found | What was done |
|---|---|---|---|
| S-1 | Art. I–III verified with `import-linter` (Python) or `NetArchTest` (.NET) | Neither tool exists for PHP | Own test, `tests/Unit/Architecture/LayersTest.php`, which reads the `use` statements and fails on a forbidden import (T-19) |
| S-2 | Contract §3.4: `PUT /books/{id}/cover` with `multipart/form-data` | **PHP does not parse multipart on a real `PUT`**: the file arrives empty | The `PUT` route is kept; the client sends `POST` + `_method=PUT`. See [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| S-3 | Laravel 11 (first choice of this translation) | Composer **blocks** every Laravel 11 version over published security advisories | Laravel 13 on PHP 8.4 was used (D-01) |
| S-4 | RNF-5: a single command | `php artisan serve` filters the environment variables, so the API ended up using SQLite | The container serves with PHP's built-in server (H-18) |
| S-5 | `library` schema with only the six tables | Laravel keeps its migration log in a `migrations` table | That table lives in the `public` schema, **outside** `library`, so the data-model §10 queries show only the six tables |

## 3. What could not be done in this delivery

| # | What the spec says | Why it could not be done | Status |
|---|---|---|---|
| N-1 | Art. IX: check with `gitleaks` or a `grep` **in CI** | The repositories have no CI, and creating one was outside the scope | Done by hand on 13-10-2026: no `.env` committed and no real secret in the six repos. `phpunit.xml` holds only dummy test values |
| N-2 | D-08: covers on **S3 in production** | The test has no production environment and no S3 account | Only the disk adapter exists. Thanks to the `CoverStorage` port, S3 would be a new adapter plus one line in `PortBindingsServiceProvider` (OCP) |
| N-3 | T-22: "manual end-to-end walkthrough **at the fair**" | The live demo takes place after the deadline | **Resolved by [ADR-011](adr/adr-011-recorded-walkthrough-as-t22-evidence.md)**: the same walkthrough is recorded with `app/e2e/walkthrough.mjs` and its screenshots. The live demo at the fair still happens |
| N-4 | Lock files (`composer.lock`, `package-lock.json`) | The team governance limits each commit to 400 lines, and these files have thousands | Not committed; dependencies are installed when the image is built. Accepted risk: exact versions are not pinned |
| N-5 | Base commit of each repo | The base commit delivered by the instructor (`ariel5253`) carries a `Co-Authored-By` trailer | Not rewritten, because it is someone else's history and would need a `push --force`. No commit in this delivery carries it |
