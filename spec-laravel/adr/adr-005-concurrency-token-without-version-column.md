# ADR-005 · The concurrency token is `available_copies`, with no `version` column

- **Status:** accepted
- **Context:** [ADR-002](../../spec-python/adr/adr-002-concurrencia-optimista.md) asks for optimistic control with a "version
  token" per book. [`data-model.md`](../../spec-python/data-model.md) §3, the source of truth for the model, has no `version`
  column. Adding one would change the data model; ignoring it would leave RNF-2 uncovered.
- **Decision:** the value read from `available_copies` is the token itself. When lending, the adapter runs
  `UPDATE library.book SET available_copies = available_copies - 1 WHERE id = :id AND available_copies = :read AND deleted_at IS NULL`.
  If it affects 0 rows, another process changed the book: the whole transaction is rolled back and the answer is
  **409 `BOOK_NOT_AVAILABLE`**. `ck_book_available_range` is still the last barrier. ADR-002's optional "retry once" is not
  implemented (see E-2 in [`../not-done-and-reasons.md`](../not-done-and-reasons.md)).
- **Rejected alternatives:**
  - adding `version integer`: it changes `data-model.md` with no requirement asking for it;
  - `SELECT … FOR UPDATE`: already rejected in ADR-002;
  - relying on the `CHECK` alone: the violation would surface as a 500 instead of a 409.
- **Consequences:** a concurrent edit of a book's total also invalidates a loan in progress. That loan fails with 409 and the
  librarian tries again. The case is rare and the outcome is safe.
- **How it is verified:** `tests/Feature/ConcurrencyTest.php` in `api`: two simultaneous loans of the last copy; exactly one
  gets 201.
