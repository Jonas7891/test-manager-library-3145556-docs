# ADR-009 · Cover routes under PHP: `GET` added from the text, `PUT` sent as `POST` + `_method`

- **Status:** accepted
- **Context:** contract §3.4 has two gaps around covers:
  1. The text says covers are drawn with `GET /books/{id}/cover`, but that route is missing from the endpoint table, so it has
     no role.
  2. `PUT /books/{id}/cover` takes `multipart/form-data`. PHP only parses multipart bodies on `POST`; on a real `PUT` the file
     arrives empty.
- **Decision:** `GET /books/{id}/cover` is served to any authenticated user (admin or librarian), as the text implies, and
  returns the binary or 404. `PUT /books/{id}/cover` stays in the routes exactly as in the contract. The React client sends
  `POST` with the field `_method=PUT`, and Laravel's method spoofing routes the request to the same `PUT` handler.
- **Rejected alternatives:** a new `POST /books/{id}/cover` route (it changes the contract); reading `php://input` and parsing
  multipart by hand (fragile code with no business value); leaving `GET` out (the frontend could not show covers).
- **Consequences:** a client that sends a literal multipart `PUT` gets a validation error, because no file arrives. This is
  documented in the `api` README.
- **How it is verified:** `test_cover_upload_validates_type_and_size_ca_03_4` (2 MB PNG accepted, 3 MB → 422, fake type → 422,
  no cover → `null`).
