# ADR-012 · Frontend performance pass and a new demo data set

- **Status:** accepted
- **Context:** the React app works (T-22), but it was written quickly. It has four weak spots:
  - it sends one request per keystroke in every search box;
  - a slow response can overwrite a newer one;
  - it loads all eight screens in one bundle, even the two that only the admin sees;
  - it repeats the same fetch logic across screens.

  Separately, the demo database should be refilled with a data set that is entirely new: books, authors, ISBNs and members.
- **Decision:**
  1. **Frontend (no new features, same screens, same contract calls)**, following the Vercel React best practices:
     - `useQuery` drops responses that arrive after a newer request (`client-` rules, no stale data);
     - search inputs use `useDeferredValue`, so typing stays responsive and requests follow the deferred value
       (`rerender-use-deferred-value`);
     - screens are loaded with `React.lazy` + `Suspense`, so the admin-only report and users screens are separate chunks
       (`bundle-dynamic-imports`, `bundle-conditional`);
     - the menu is filtered with a precomputed list per role (`rendering-hoist-jsx`, `js-early-exit`).
  2. **Demo data:** `tool/seed/data/books.json` and `members.json` are replaced with 40 different books (same five
     categories, same copy counts per position, so the loan plan stays valid) and 10 different members. ISBNs and document
     numbers start from new ranges. The seeder picks the overdue books by position, not by hard-coded title. The database is
     emptied with `docker compose down -v` and refilled **only through the API** (`data-model.md` §9). The schema is not
     touched.
- **Rejected alternatives:**
  - Next.js / server components: they change the stack defined in `plan.md`;
  - SWR or React Query: a new dependency for something 20 lines solve;
  - loading the data with SQL: forbidden by data-model §9.
- **Consequences:** the T-21 and T-22 evidence is regenerated with the new data.
- **How it is verified:** `npm run build` shows separate chunks; `node seed/seed.mjs --check` prints "Vencidos: 3" with the new
  titles; `npm run walkthrough` passes all 8 steps again.
- **Result (13-10-2026):**
  - `npm run build` emits one chunk per screen; `ReportPage` and `UsersPage` are loaded only by the admin.
  - The database was emptied and reseeded through the API. `seed.mjs --check` → "Vencidos: 3" (Charlie y la fábrica de
    chocolate, Crónica de una muerte anunciada, El hombre que confundió a su mujer con un sombrero) and 12 report rows.
  - `npm run walkthrough`: 8 steps green; screenshots refreshed in `app/evidence/t-22/`.
