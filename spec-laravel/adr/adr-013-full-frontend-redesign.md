# ADR-013 · Full frontend redesign with the same behaviour and the same API calls

- **Status:** accepted
- **Context:** the owner asked for the React frontend to be replaced completely: a new look, layout and component structure,
  while every function and every API call stays exactly as it is today (T-22, `api-contract.md` §3).
- **Decision:**
  - **What changes completely:**
    - *Layout:* a fixed side navigation replaces the top bar, and a two-column login replaces the centred card.
    - *Design system:* a new `theme.css` with colour tokens and light and dark mode replaces `styles.css`.
    - *Structure:* `pages/` and `components/Common.jsx` are replaced by `screens/` (one file per screen), `ui/` (reusable
      blocks: `Shell`, `PageHeader`, `Alert`, `DataTable`, `Pager`, `Field`) and `lib/` (`useQuery`, formatters).
    - *Screen markup:* every screen gets a page header with a description, and tables gain empty states.
  - **What does not change:**
    - `api/client.js` and `auth/AuthContext.jsx`, so the requests (method, path, query, body, `_method=PUT` for covers) are
      byte-for-byte the same;
    - roles and menus (the librarian does not see the report or users screens);
    - the validations and messages that come from the API;
    - the accessible names of links, headings, fields and buttons. This lets `e2e/walkthrough.mjs` (ADR-011) act as the
      regression check for the redesign.
  - **What is not added:** no new screens, filters, endpoints or dependencies.
- **Rejected alternatives:**
  - a UI kit such as MUI or Tailwind: a new dependency and a heavier bundle;
  - rewriting the HTTP client: it would put "same API calls" at risk with no gain.
- **Consequences:** the T-22 screenshots are taken again with the new design.
- **How it is verified:**
  - `npm run build` passes;
  - `npm run walkthrough` passes all 8 steps;
  - `git diff` shows no change to `src/api/client.js` or `src/auth/AuthContext.jsx`.
- **Result (13-10-2026):**
  - `src/pages/`, `src/components/` and `styles.css` were replaced by `screens/`, `ui/`, `lib/` and `theme.css`;
  - `git diff` on `src/api/client.js` and `src/auth/AuthContext.jsx` is empty;
  - `npm run build` passes;
  - `npm run walkthrough`: 8 steps green on the rebuilt container; screenshots refreshed in `app/evidence/t-22/`.
