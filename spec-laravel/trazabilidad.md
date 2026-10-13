# Trazabilidad — de la historia a la prueba

Cada criterio de aceptación de [`../spec-python/spec.md`](../spec-python/spec.md) se rastrea hasta el endpoint, la clase que lo
cumple (repo `api`, salvo indicación) y la prueba que lo demuestra.

| HU / CA | Endpoint | Dónde se cumple | Prueba |
|---|---|---|---|
| CA-01.1 token de 8 h | `POST /auth/login` | `LoginService`, `JwtTokenIssuer` | `test_login_hu_01` |
| CA-01.2 401 sin decir qué falló | `POST /auth/login` | `LoginService` → `InvalidCredentials` | `test_login_hu_01`, `UseCasesTest::test_login_*` |
| CA-01.3 401 sin token / 403 por rol | todos | `JwtAuth`, `RequireRole` | `test_token_and_role_are_enforced_ca_01_3` |
| CA-02.1 / 02.2 alta y normalización | `POST /users` | `User::createLibrarian` | `test_create_librarian_hu_02`, `MemberAndUserTest` |
| CA-02.3 `admin` por API → 422 | `POST /users` | `User::createLibrarian` | `test_create_librarian_hu_02` |
| CA-03.1 disponibles = total | `POST /books` | `Book::create` | `BookTest`, `test_books_crud_and_search_hu_03_hu_04` |
| CA-03.2 ISBN 13 dígitos y único | `POST /books` | `Isbn`, índice `ux_book_isbn_active` | `ValueObjectsTest`, `test_books_crud_*` |
| CA-03.3 total ≥ 1 y diferencia | `PUT /books/{id}` | `Book::update`, `ck_book_total_positive` | `BookTest`, `DatabaseInvariantsTest` |
| CA-03.4 portada JPG/PNG ≤ 2 MB | `PUT /books/{id}/cover` | `BookCoverService` | `test_cover_upload_validates_type_and_size_ca_03_4` |
| CA-03.5 baja lógica | `DELETE /books/{id}` | `Book::delete`, FK `RESTRICT` | `test_lending_rules_*`, `DatabaseInvariantsTest` |
| CA-04.1 / 04.2 búsqueda paginada | `GET /books` | `PgBookRepository::search` | `test_books_crud_and_search_hu_03_hu_04` |
| CA-04.3 404 dado de baja | `GET /books/{id}` | `PgBookRepository::find` | `test_books_crud_and_search_hu_03_hu_04` |
| CA-05.1 / 05.2 / 05.3 socios | `/members` | `MembersService` (sin DELETE) | `test_members_hu_05` |
| CA-06.1 1 a 3 líneas | `POST /loans` | `Loan::open` | `LoanTest`, `test_lending_rules_*` |
| CA-06.2 sin libro repetido | `POST /loans` | `Loan::open` | `test_lending_rules_ca_06_2_ca_06_4` |
| CA-06.3 descuenta en la misma operación | `POST /loans` | `Book::withdraw` + `PgLoanRepository::saveNew` | `test_lend_and_return_hu_06_hu_07`, `ConcurrencyTest` |
| CA-06.4 máximo 3 con conteo | `POST /loans` | `Loan::open` → `MemberLimitExceeded` | `test_lending_rules_ca_06_2_ca_06_4` |
| CA-06.5 socio con vencidos | `POST /loans` | `Loan::open` → `MemberHasOverdue` | `test_overdue_blocks_and_lists_ca_06_5_hu_09` |
| CA-06.6 14 días y valores congelados | `POST /loans` | `Loan::addItem` | `test_lend_and_return_hu_06_hu_07`, `LoanTest` |
| CA-07.1 / 07.2 devolución por línea, una vez | `POST /loans/{id}/items/{item}/return` | `LoanItem::markReturned`, `PgLoanRepository::markReturned` | `test_lend_and_return_hu_06_hu_07` |
| CA-07.3 disponibles ≤ total | — | `ck_book_available_range` | `ConcurrencyTest::test_database_check_is_the_last_barrier` |
| CA-07.4 / 08.1 / 08.3 estado calculado | `GET /loans/{id}` | `Loan::status` | `LoanTest::test_status_is_calculated_*` |
| CA-08.2 listado con filtros | `GET /loans` | `PgLoanRepository::search` (H-10) | `test_overdue_blocks_and_lists_*` |
| CA-09.1 / 09.2 vencidos | `GET /loans/overdue` | `PgLoanRepository::overdue` | `test_overdue_blocks_and_lists_ca_06_5_hu_09` |
| CA-10.1 / 10.4 reporte congelado | `GET /reports/most-borrowed` | `PgReportReader` (ADR-004) | `test_report_is_frozen_and_admin_only_hu_10` |
| CA-10.2 rango invertido / vacío | `GET /reports/most-borrowed` | `DateRange` | `test_report_is_frozen_*`, `ValueObjectsTest` |
| CA-10.3 / 10.5 sin socios, solo admin | `GET /reports/most-borrowed` | `PgReportReader`, `role:admin` | `test_report_is_frozen_*`, `test_token_and_role_*` |

## Frontend (`app`)

| HU | Pantalla |
|---|---|
| HU-01 | `LoginPage.jsx` |
| HU-02 | `UsersPage.jsx` |
| HU-03 / HU-04 | `BooksPage.jsx` |
| HU-05 | `MembersPage.jsx` |
| HU-06 | `NewLoanPage.jsx` |
| HU-07 / HU-08 | `LoansPage.jsx` |
| HU-09 | `OverduePage.jsx` |
| HU-10 | `ReportPage.jsx` |
