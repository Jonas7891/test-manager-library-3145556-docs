# Audit of the base commit: inconsistencies that prevent a 100 % literal completion

This audit covers the instructor's base commit `d1364dd` ("Prueba técnica SDD ficha 3145556: Manager Library (docs)", author
`ariel5253`, 13-10-2026). It compares `spec-python/` with `spec-.net/` and checks each document against the others.

**The base was not corrected.** `spec-python/`, `spec-.net/` and the root `README.md` are byte-for-byte the same as in `d1364dd`
(`git diff --stat d1364dd HEAD -- spec-python spec-.net README.md` prints nothing). Each finding is recorded here. Where the
Laravel translation had to take a different path to keep working, the change lives only in `spec-laravel/` and is backed by an
ADR. Where no workaround was possible, the item stays open and is marked **not done**.

## 1. `spec-python/` vs `spec-.net/`

Running `diff -r spec-python spec-.net` shows that the two folders differ only in technology. Business rules, user stories, the
HTTP contract and the data model are identical.

| # | Difference | Files | Assessment |
|---|---|---|---|
| P-1 | Framework, ORM, migrations, test runner and folder paths (`src/library/` vs `src/Library/`) | `plan.md` D-01–D-03, D-12; `architecture.md`; `constitution.md` | Expected: this is the stack the reference spec targets |
| P-2 | Password hash: **Argon2id** (Python) vs **BCrypt factor 12** (.NET) | `plan.md` D-09 | Not a business conflict, but the challenge stack is neither. The translation follows Python (Argon2id) |
| P-3 | Architecture check: `import-linter` + `tests/architecture/` vs **NetArchTest** | `constitution.md` Art. I–II, `plan.md` §3, `tasks.md` T-19 | Neither tool exists for PHP. See S-1 in [`not-done-and-reasons.md`](not-done-and-reasons.md) |

## 2. Inconsistencies inside the base spec (both versions)

| # | What the base says | Inconsistency | Status in this delivery | Backed by |
|---|---|---|---|---|
| B-1 | ADR-002, D-04 and T-13: an optimistic **"version token" per book** | `data-model.md` §3, which says it wins over any other document, has **no `version` column** | Base not corrected. The translation uses `available_copies` as the token | [ADR-005](adr/adr-005-concurrency-token-without-version-column.md) |
| B-2 | ADR-002: on conflict return 409 **"or retry once"** | Two allowed outcomes; the contract only documents 409 | **Not done**: the retry was not implemented. Always 409 `BOOK_NOT_AVAILABLE` | [ADR-005](adr/adr-005-concurrency-token-without-version-column.md) |
| B-3 | T-21: the seeder creates **3 overdue loans through the API** | Impossible as written: HU-06 makes the server set `loaned_at`, and the contract accepts no date. A loan made today is not overdue for 14 days | Workaround in the translation only (a shiftable clock, 0 by default) | [ADR-006](adr/adr-006-shiftable-clock-for-the-seeder.md) |
| B-4 | Contract §1 and CA-01.3: 401 with no token or an expired token | Catalogue §2 has **no error code** for that case | The only 401 code is reused with another message | [ADR-008](adr/adr-008-reuse-invalid-credentials-for-missing-token.md) |
| B-5 | T-01 and CA-01.3: `GET /health`; contract §1: base path `/api/v1` | The two paths do not match | Both paths are served | [`decisions-and-gaps.md`](decisions-and-gaps.md) H-02 |
| B-6 | Contract §3.4: `GET /books/{id}/cover` appears in the text | Missing from the endpoint table, so it has no role | Implemented for any authenticated user | [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| B-7 | Contract §3.4: `PUT /books/{id}/cover` with `multipart/form-data` | Valid HTTP, but PHP does not parse multipart bodies on a real `PUT` | Route kept; the client sends `POST` + `_method=PUT` | [ADR-009](adr/adr-009-cover-routes-in-php.md) |
| B-8 | Contract §3.6: `/loans/overdue` and `/loans/{id}` | `overdue` can be read as an id | `overdue` is declared first and `{id}` only accepts a UUID | H-04 |
| B-9 | T-18: "`data-model.md` §4 is updated" once the constraints are applied | The base `data-model.md` keeps them as **pending (T-18)**. Editing the base would rewrite the instructor's spec | Constraints applied in the migrations; the base was not edited. The new marks live in the translation | [ADR-010](adr/adr-010-t18-constraints-recorded-in-the-translation.md) |
| B-10 | Constitution Art. IX: secret check with `gitleaks` or `grep` **in CI** | The six repositories have no CI, and adding it is outside the scope | **Not done in CI**; a manual check was done on 13-10-2026 | [`not-done-and-reasons.md`](not-done-and-reasons.md) N-1 |
| B-11 | D-08: covers on S3 in production | There is no production environment or S3 account for the test | **Not done**; disk adapter only, behind the `CoverStorage` port | N-2 |
| B-12 | T-22: evidence is "a manual end-to-end walkthrough **at the fair**" | That evidence only exists after the deadline | **Partial**: the build works and the walkthrough is written down | N-3 |
| B-13 | `plan.md` D-07 lists four value objects (`Isbn`, `Quantity`, `DocumentNumber`, `DateRange`) | T-03 lists only three and leaves out `DocumentNumber` | `DocumentNumber` was implemented and tested anyway (`ValueObjectsTest`) | — |
| B-14 | `spec-python/tasks.md` T-04/T-05 use method names `Withdraw`, `Restock`, `AddItem`, `ReturnItem` | PascalCase is the .NET convention, copied into the Python version | Naming only; the translation uses PHP camelCase (`withdraw`, `restock`) | — |
| B-15 | Base commit message footer | The instructor's commit `d1364dd` (and the base commit of the other five repositories) carries a `Co-Authored-By` trailer, which the delivery rules forbid for our commits | **Not changed**: it is someone else's history and would need a `push --force` | N-5 |

## 3. Summary

- **Corrected in the translation and backed by an ADR:** B-1, B-3, B-4, B-6, B-7 and B-9.
- **Resolved as a gap decision (no architecture impact):** B-5, B-8, B-13 and B-14.
- **Not done, with the reason recorded:** B-2, B-10, B-11, B-12 and B-15.
- **Base spec:** unchanged. These small faults are left in the base as delivered.
