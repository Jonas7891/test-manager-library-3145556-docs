# ADR-014 · New demo credentials, with passwords kept out of the repositories

- **Status:** accepted
- **Context:** the owner asked to replace every demo credential:
  - admin: user `Jonas7891`;
  - librarian: user `Juan Diego`.

  Until now the seeder and the walkthrough script carried the old librarian password as a default value in the code. Article IX
  forbids versioned passwords.
- **Decision:**
  - **Usernames** are not secrets, so they become the defaults:
    - `ADMIN_USERNAME=Jonas7891` in the `infra`, `api` and `tool` `.env.example` files;
    - `LIBRARIAN_USERNAME`, default `Juan Diego`, in `tool` and in `app/e2e`.
    - The API stores them normalised (trimmed and lower-cased, CA-02.1): `jonas7891` and `juan diego`. Login applies the same
      rule, so both spellings work.
  - **Passwords** never go into a versioned file:
    - `ADMIN_PASSWORD` and `LIBRARIAN_PASSWORD` live only in the local `infra/.env` (git-ignored) and are passed as environment
      variables;
    - the old hard-coded librarian password is removed, and the scripts now stop with an error if the variable is missing.
  - The demo database is emptied (`docker compose down -v`) and seeded again through the API, so the old `admin` and
    `bibliotecario.demo` users no longer exist.
- **Rejected alternatives:**
  - writing the new passwords as defaults in the code: this breaks Article IX;
  - keeping the old users next to the new ones: the request is a complete change.
- **How it is verified:**
  - `git grep` finds neither password in any of the six repositories;
  - logging in with the new admin and librarian works;
  - `npm run walkthrough` passes with the new users.
- **Result (13-10-2026):**
  - the database was reset and reseeded;
  - logins: `Jonas7891` → 200, `Juan Diego` → 200, old `admin` → 401, old `bibliotecario.demo` → 401;
  - `git grep` on the six repositories finds no password;
  - the walkthrough is green with the new users.
