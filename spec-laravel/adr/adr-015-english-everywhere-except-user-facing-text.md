# ADR-015 · English everywhere except text shown to users

- **Status:** accepted (being applied)
- **Context:** the owner asked that all documentation, code and comments be in English, and that only the messages users see
  stay in Spanish.
- **Decision:**
  - **In English:** `spec-laravel/`, every repository README, code comments, commit messages, and the comments in
    `.env.example` and Docker files.
  - **In Spanish (user-facing):**
    - API error messages (`message` in the error body);
    - React screen texts and labels;
    - the static `page` site;
    - seeder console output, demo data and the base spec written by the instructor.
- **Inventory on 13-10-2026 at 16:54:**
  - `spec-laravel/`: already in English (ADR-013 and earlier);
  - READMEs still in Spanish: `api`, `app`, `infra`, `tool` and `page`;
  - code comments still in Spanish: about 91 lines in `api`, 16 in `app`, 10 in `tool` and 5 in `infra`.
- **How it is verified:** a `git grep` for Spanish words in comments and READMEs returns only user-facing strings.
