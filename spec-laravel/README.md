# spec-laravel — Manager Library translated to React + Laravel

This folder **does not rewrite** the spec. `spec-python/` and `spec-.net/` remain the business source of truth (user stories,
acceptance criteria, rules, HTTP contract and data model). This folder records **how** that spec was carried over to the
challenge stack, and the evidence behind each task marked as done.

## How the spec was read

1. `constitution.md`: the 12 articles are kept; only the tool that verifies them changes.
2. `spec.md`, `data-model.md`, `api-contract.md`: followed **unchanged**, since they do not depend on the language.
3. `plan.md` / `architecture.md`: translated to Laravel in [`plan.md`](plan.md) and [`architecture.md`](architecture.md).
4. `tasks.md`: T-01…T-23 are reused with their evidence. The real status is in [`tasks.md`](tasks.md).
5. Where the spec is ambiguous or contradicts itself:
   - the decision is in [`decisions-and-gaps.md`](decisions-and-gaps.md);
   - if it affects the architecture, there is also a new ADR (`adr/adr-005` onwards);
   - every fault found in the base commit is listed in [`base-spec-audit.md`](base-spec-audit.md).

`spec-python/` and `spec-.net/` differ only in framework, ORM, hash algorithm, architecture test tool and folder paths; the
business is identical. `spec-python/` was used as the reading base.

## What changes compared with Python / .NET

| Topic | Python | .NET | Challenge (this document) |
|---|---|---|---|
| Backend | FastAPI | ASP.NET Core | **Laravel 13 (PHP 8.4)** |
| Persistence | SQLAlchemy + Alembic | EF Core + migrations | **Laravel migrations + Query Builder** |
| Hash | Argon2id | BCrypt | **Argon2id** (`password_hash`) |
| Token | JWT HS256 | JWT HS256 | **JWT HS256** with `firebase/php-jwt` ([ADR-007](adr/adr-007-own-jwt-instead-of-sanctum.md)) |
| Tests | pytest | xUnit | **PHPUnit** |
| Architecture | import-linter | NetArchTest | **PHPUnit test that inspects `use` statements** (`tests/Unit/Architecture`) |
| Frontend | React | React | **React + Vite** |
| Engine | PostgreSQL 16 | PostgreSQL 16 | **PostgreSQL 16** (no change) |

## Index

| File | Content |
|---|---|
| [`plan.md`](plan.md) | Technical decisions D-01…D-12, translated |
| [`architecture.md`](architecture.md) | Hexagonal layers inside Laravel and the real folders |
| [`decisions-and-gaps.md`](decisions-and-gaps.md) | Spec ambiguities and the decision taken for each |
| [`traceability.md`](traceability.md) | User story / acceptance criterion → endpoint → class → test |
| [`tasks.md`](tasks.md) | Real status of T-01…T-23 with a link to the evidence |
| [`base-spec-audit.md`](base-spec-audit.md) | Inconsistencies found in the instructor's base commit (base left unchanged) |
| [`not-done-and-reasons.md`](not-done-and-reasons.md) | What could not be done as the spec says, and why |
| [`adr/`](adr/) | ADR-005 to ADR-013 |
| `evidence/` | Command outputs used as evidence |

## Repositories

| Repo | Content |
|---|---|
| `test-manager-library-3145556-api` | Laravel backend |
| `test-manager-library-3145556-app` | React frontend |
| `test-manager-library-3145556-infra` | `docker compose` with an empty engine |
| `test-manager-library-3145556-tool` | Demo seeder that uses the API |
| `test-manager-library-3145556-page` | Static public site |
