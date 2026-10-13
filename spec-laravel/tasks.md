# Tasks — Manager Library (Laravel + React): real status

Same tasks as [`../spec-python/tasks.md`](../spec-python/tasks.md). A task is **done** only if its evidence exists and can be run
(article XII). Possible states: **done**, **partial** (part of the evidence is missing) and **pending**.

Paths are relative to each repository. The `api` suite runs with
`docker compose -f docker-compose.test.yml run --rm --build tests` and gives **59 tests, 394 assertions, all green**
(13-10-2026).

| # | Task | Status | Evidence |
|---|---|---|---|
| T-01 | Skeleton and `GET /health` | done | `api`: `AuthAndCatalogContractTest::test_health_is_public_t_01`; `curl localhost:8000/api/v1/health` → `{"status":"ok"}` |
| T-02 | Initial migration with seed | done | `api`: `database/migrations/2026_10_13_000002_*`; data-model §10 queries in [`evidence/t-02-schema.txt`](evidence/t-02-schema.txt) (no column with `DEFAULT`); `DatabaseInvariantsTest::test_no_column_has_a_default_and_there_is_no_status_column` |
| T-03 | Value objects | done | `api`: `tests/Unit/Domain/ValueObjectsTest.php` |
| T-04 | `Book` | done | `api`: `tests/Unit/Domain/BookTest.php` (never < 0, never > total) |
| T-05 | `Loan` | done | `api`: `tests/Unit/Domain/LoanTest.php`, with a fixed clock: `open`, `overdue`, `closed` |
| T-06 | `Member` and `User` | done | `api`: `MemberAndUserTest` (`"Ana "` → `ana`; role `admin` rejected) |
| T-07 | Authentication with fakes | done | `api`: `UseCasesTest::test_login_*`, `test_librarian_creation_rules_hu_02` |
| T-08 | Books with fakes | done | `api`: `UseCasesTest::test_isbn_unique_*`, `test_cannot_delete_book_with_open_loans_ca_03_5` |
| T-09 | Lending with fakes | done | `api`: `UseCasesTest::test_lend_*`, `test_member_with_overdue_is_blocked_ca_06_5` |
| T-10 | Returns and overdue | done | `api`: `UseCasesTest::test_return_restocks_once_and_twice_fails_hu_07`; `LoansContractTest::test_overdue_blocks_and_lists_*` |
| T-11 | Most-borrowed report | done | `api`: `UseCasesTest::test_report_keeps_frozen_title_after_rename_adr_004`; `LoansContractTest::test_report_is_frozen_and_admin_only_hu_10` |
| T-12 | Book, member and user repositories | done | `api`: `tests/Feature/*ContractTest` against a real PostgreSQL |
| T-13 | Loan repository + concurrency token | done | `api`: `tests/Feature/ConcurrencyTest.php`: only one loan is recorded for the last copy ([ADR-005](adr/adr-005-concurrency-token-without-version-column.md)) |
| T-14 | Hash and JWT | done | `api`: the contract checks that no response contains `password`; Argon2id + 8-hour HS256 JWT (`expires_in: 28800`) |
| T-15 | Covers on disk | done | `api`: `test_cover_upload_validates_type_and_size_ca_03_4`: 2 MB PNG accepted, 3 MB → 422, fake type → 422, no cover → `null` |
| T-16 | HTTP adapter and single error body | done | `api`: `AuthAndCatalogContractTest` and `LoansContractTest` cover the routes and the §2 codes |
| T-17 | Composition and initial admin | done | `api`: `PortBindingsServiceProvider`; `library:bootstrap-admin` at startup; admin login on a freshly created database |
| T-18 | `ck_book_total_positive`, `ck_user_role` | done | `api`: migration `2026_10_13_000003_*`; `DatabaseInvariantsTest`; listed in [`evidence/t-02-schema.txt`](evidence/t-02-schema.txt). The **engine** mark is recorded here, not in the base `data-model.md` ([ADR-010](adr/adr-010-t18-constraints-recorded-in-the-translation.md)) |
| T-19 | Architecture tests | done | `api`: `tests/Unit/Architecture/LayersTest.php` (includes a test showing the checker catches forbidden imports) |
| T-20 | `infra`: `docker compose up` | done | `infra`: `docker compose up -d --build` starts db, api and app; the repo contains no `CREATE TABLE` |
| T-21 | `tool`: seeder through the API | done | `tool`: `node seed/seed.mjs --check` → "Vencidos: 3" and 12 report rows (output in the `tool` README) |
| T-22 | `app`: end-to-end walkthrough | done | `app`: `npm run walkthrough` drives the real app in a browser through 8 steps (login, catalogue, members, lend, return, overdue, report, librarian menus). The 13-10-2026 run was all green; screenshots in `evidence/t-22/` ([ADR-011](adr/adr-011-recorded-walkthrough-as-t22-evidence.md)). The live walkthrough at the fair repeats it |
| T-23 | `page`: static site | done | `page`: HTML and CSS without JavaScript (`grep -c "fetch\|<script" index.html` → 0) and a responsive layout |

## Non-functional requirements

| # | Status | Evidence |
|---|---|---|
| RNF-1 (search < 300 ms with 5,000 books) | done | `tool`: `node seed/load.mjs` with 5,000 books: p95 58.3 ms, max 66.8 ms |
| RNF-2 (last copy) | done | `ConcurrencyTest` |
| RNF-3 (password never in plain text or logs) | done | Argon2id; `assertErrorBody` checks that no response contains `password`; domain errors are not logged |
| RNF-4 (single error body) | done | `ErrorRenderer`; every contract test uses `assertErrorBody` |
| RNF-5 (one command, empty database) | done | `infra` README |
