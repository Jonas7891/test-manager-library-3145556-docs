# ADR-007 · JWT with `firebase/php-jwt` instead of Sanctum

- **Status:** accepted
- **Context:** the contract (§4) and D-11 ask for a stateless **HS256-signed JWT**, valid for 8 hours, with the `sub` and `role`
  claims. Laravel's default option, Sanctum, issues opaque tokens stored in a `personal_access_tokens` table. That table does not
  exist in `data-model.md`.
- **Decision:** the `TokenIssuer` port is implemented by `JwtTokenIssuer` with `firebase/php-jwt` and the `JWT_SECRET` key. The
  `JwtAuth` middleware checks the signature and expiry, and `RequireRole` checks the role in the claim.
- **Rejected alternatives:** Sanctum (it adds a table, and its token is not a JWT); `tymon/jwt-auth` (a large package coupled to
  Eloquent).
- **Consequences:** a token cannot be revoked before it expires. The original spec has the same limit, because a JWT holds no
  state.
- **How it is verified:** the contract tests check that a request without a token gets 401, that a `librarian` token on
  `/reports/most-borrowed` gets 403, and that `expires_in` is 28800.
