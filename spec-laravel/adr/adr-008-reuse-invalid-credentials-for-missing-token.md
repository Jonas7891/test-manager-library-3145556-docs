# ADR-008 · A missing or expired token reuses `INVALID_CREDENTIALS`

- **Status:** accepted
- **Context:** contract §1 and CA-01.3 require **401** when the token is missing, invalid or expired. The error catalogue (§2)
  has only one 401 code, `INVALID_CREDENTIALS`, and ties it to CA-01.2 (wrong username or password). The base spec defines no
  code for the token case, and RNF-4 forbids responses outside the `{code, message, details}` body.
- **Decision:** the `JwtAuth` middleware answers 401 with `code: "INVALID_CREDENTIALS"` and a different message, "Se requiere un
  token válido y vigente.". No new code is added to the catalogue.
- **Rejected alternatives:** a new code such as `TOKEN_INVALID` (it widens the contract, and no requirement asks for it);
  Laravel's default `{"message":"Unauthenticated."}` body (it breaks RNF-4).
- **Consequences:** clients cannot tell "bad login" from "bad token" by the code alone. The React app handles both the same way:
  it clears the session and goes back to the login page.
- **How it is verified:** `AuthAndCatalogContractTest::test_token_and_role_are_enforced_ca_01_3` in `api` (no token → 401 with
  the single error body).
