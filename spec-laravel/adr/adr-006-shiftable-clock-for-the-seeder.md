# ADR-006 · Clock shift for the demo only

- **Status:** accepted
- **Context:** T-21 asks the seeder to create loans **through the API**, so that `/loans/overdue` then returns 3 lines.
  However, the server sets `loaned_at` with its own clock (HU-06), and the contract does not accept a date. As a result, no loan
  created today can be overdue before 14 days have passed.
- **Decision:** the `SystemClock` adapter adds `CLOCK_OFFSET_DAYS` days to the real time.
  - The default is `0`, and the setting is not part of the HTTP contract.
  - It is changed only when starting the API for the seeder's "past" phase, for example to `-20`.
  - The domain is unaware of it: it already receives the time through the `Clock` port (D-05).
- **Rejected alternatives:**
  - inserting loans with SQL: `data-model.md` §9 forbids it;
  - a `loaned_at` field in `POST /loans`: it widens the contract and opens an integrity hole;
  - a header that changes the time per request: any client could fake dates.
- **Consequences:** with a non-zero shift the whole system runs on another date. That is why it is used only for the demo and is
  never set in `.env.example` to anything but `0`.
- **How it is verified:** the `tool` README shows both phases. After running them, `GET /loans/overdue` returns 3 lines.
