# Technical plan — Manager Library (Laravel + React)

Translation of [`../spec-python/plan.md`](../spec-python/plan.md). The business does not change; only the technology does.

## 1. Technical decisions

| # | Decision | Choice (challenge) | Why |
|---|---|---|---|
| D-01 | Language and runtime | PHP 8.4 + Laravel 13 | Challenge stack. Laravel 11 was dropped because Composer blocks all its versions over published security advisories |
| D-02 | HTTP framework | Laravel routes with the `api/v1` prefix, thin controllers | Contract in `api-contract.md` |
| D-03 | Persistence | Laravel migrations (explicit SQL) + Query Builder on PostgreSQL 16, schema `library` | ADR-001: the schema belongs to the migrations |
| D-04 | Concurrency | Compare-and-set on `available_copies` + `CHECK ck_book_available_range` ([ADR-005](adr/adr-005-concurrency-token-without-version-column.md)) | RNF-2 without adding columns to the model |
| D-05 | Dates | `timestamptz`, application in UTC; the clock is the `Clock` port | Test due dates without waiting 14 days |
| D-06 | Report | One aggregate query behind the `ReportReader` port | ADR-004 |
| D-07 | Value objects | `final readonly` PHP classes: `Isbn`, `Quantity`, `DocumentNumber`, `DateRange` | No table, no identity |
| D-08 | Covers | Local disk (`COVERS_DIR`) behind `CoverStorage`; the database stores only `cover_key` | Opaque key |
| D-09 | Passwords | Argon2id (`password_hash`) behind `PasswordHasher` | Article IX |
| D-10 | Startup | `php artisan library:bootstrap-admin` command, from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot produce the hash |
| D-11 | Authentication | JWT HS256, 8 h, `firebase/php-jwt`, key `JWT_SECRET` ([ADR-007](adr/adr-007-own-jwt-instead-of-sanctum.md)) | Required by contract §4 |
| D-12 | Tests | PHPUnit: pure domain, application with fakes, contract against a real PostgreSQL | Test pyramid |

## 2. Configuration by environment

| Variable | Required | Example |
|---|---|---|
| `DB_HOST` / `DB_PORT` / `DB_DATABASE` / `DB_USERNAME` / `DB_PASSWORD` | yes | `db` / `5432` / `library` / `library` / *(in `.env`)* |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | set in `.env`, never in the repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |
| `CLOCK_OFFSET_DAYS` | no | `0` (see [ADR-006](adr/adr-006-shiftable-clock-for-the-seeder.md)) |

## 3. Test strategy

| Level | What it tests | Where (`api` repo) |
|---|---|---|
| Domain | Invariants: at most 3 loans, due in 14 days, available copies in range | `tests/Unit/Domain/` |
| Engine | `CHECK`, FK and no column `DEFAULT` | `tests/Feature/DatabaseInvariantsTest.php` |
| Application | Use cases with fake ports | `tests/Unit/Application/` |
| Contract | Routes, §2 codes, single error body | `tests/Feature/` (real PostgreSQL) |
| Architecture | Articles I, II and III | `tests/Unit/Architecture/` |
| Concurrency | Two loans of the last copy | `tests/Feature/ConcurrencyTest.php` |

## 4. Build order

Same as the spec:
1. skeleton and health check;
2. initial migration with the seed;
3. domain;
4. use cases with fakes;
5. adapters;
6. API;
7. contract and concurrency tests;
8. infra, tool, app and page.
