# ADR-011 · T-22 evidence: a recorded end-to-end walkthrough before the fair

- **Status:** accepted
- **Context:** T-22 (`app`: login, catalogue, members, lend, return, overdue, report) gives "a manual end-to-end walkthrough at
  the fair" as its evidence. The fair takes place after the deadline, so on delivery day that evidence cannot exist. Article XII
  says a task is done only when its evidence exists and can be run again.
- **Decision:** the same walkthrough is run now and recorded, without adding any feature to the app.
  - `app/e2e/walkthrough.mjs` drives the real React app in a browser, against the stack started by `infra` and seeded by `tool`.
  - It follows the steps of the `app` README:
    1. log in as admin;
    2. open the catalogue and the members list;
    3. register a loan;
    4. return one line;
    5. open the overdue list and the report;
    6. log in as `bibliotecario.demo` and check that the admin-only menus are hidden.
  - It saves one screenshot per step in `app/evidence/t-22/`, and stops with exit code 1 if any step fails.
  - It uses `playwright-core` with the browser already installed on the machine (Edge or Chrome), so no browser is downloaded.
    It is a development dependency only, and does not ship in the image.
  - The live walkthrough at the fair still happens; it repeats this script by hand.
- **Rejected alternatives:**
  - waiting for the fair: T-22 would stay partial at the deadline;
  - screenshots taken by hand with no script: they cannot be run again, which Article XII requires;
  - a full test framework such as Cypress or the Playwright test runner: it is more than the evidence needs.
- **Consequences:** the walkthrough changes data (one loan and one return) on the demo database, the same as a manual run would.
- **How it is verified:** `node e2e/walkthrough.mjs` in `app` prints each step as `ok`, and `app/evidence/t-22/` holds the
  screenshots.
