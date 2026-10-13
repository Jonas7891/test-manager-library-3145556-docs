# ADR-010 · T-18 constraints are recorded in the translation, not in the base `data-model.md`

- **Status:** accepted
- **Context:** T-18 moves `ck_book_total_positive` and `ck_user_role` into the database, and its evidence says
  "`data-model.md` §4 is updated". The base `data-model.md` is part of the instructor's commit `d1364dd`, and this delivery does
  not rewrite the base spec (see [`../base-spec-audit.md`](../base-spec-audit.md)).
- **Decision:** the two constraints are created by migration `2026_10_13_000003_*` in `api`. The base `data-model.md` §4 is
  **left as delivered** (marked "pending (T-18)"). The new **engine** mark is recorded in this translation: [`../tasks.md`](../tasks.md)
  T-18 and the schema evidence [`../evidence/t-02-schema.txt`](../evidence/t-02-schema.txt).
- **Rejected alternatives:** editing `spec-python/data-model.md` and `spec-.net/data-model.md` (it changes the instructor's
  source of truth); skipping T-18 (it lowers the delivered scope).
- **Consequences:** a reader of the base spec still sees "pending". This ADR and the audit explain the difference.
- **How it is verified:** `DatabaseInvariantsTest` in `api`: an `INSERT` that breaks either rule fails in the engine.
