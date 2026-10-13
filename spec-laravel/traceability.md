# Traceability — from user story to test

Each acceptance criterion in [`../spec-python/spec.md`](../spec-python/spec.md) is traced to its endpoint, the class that fulfils it
(`api` repo unless stated otherwise) and the test that proves it.

| Story / criterion | Endpoint | Fulfilled in | Test |
|---|---|---|---|
| CA-01.1 8-hour token | `POST /auth/login` | `LoginService`, `JwtTokenIssuer` | `test_login_hu_01` |
| CA-01.2 401 without saying what failed | `POST /auth/login` | `LoginService` → `InvalidCredentials` | `test_login_hu_01`, `UseCasesTest::test_login_*` |
| CA-01.3 401 without token / 403 by role | all | `JwtAuth`, `RequireRole` | `test_token_and_role_are_enforced_ca_01_3` |
| CA-02.1 / 02.2 creation and normalization | `POST /users` | `User::createLibrarian` | `test_create_librarian_hu_02`, `MemberAndUserTest` |
| CA-02.3 `admin` via API → 422 | `POST /users` | `User::createLibrarian` | `test_create_librarian_hu_02` |
| CA-03.1 available = total | `POST /books` | `Book::create` | `BookTest`, `test_books_crud_and_search_hu_03_hu_04` |
| CA-03.2 13-digit unique ISBN | `POST /books` | `Isbn`, index `ux_book_isbn_active` | `ValueObjectsTest`, `test_books_crud_*` |
| CA-03.3 total ≥ 1 and difference | `PUT /books/{id}` | `Book::update`, `ck_book_total_positive` | `BookTest`, `DatabaseInvariantsTest` |
| CA-03.4 JPG/PNG cover ≤ 2 MB | `PUT /books/{id}/cover` | `BookCoverService` | `test_cover_upload_validates_type_and_size_ca_03_4` |
| CA-03.5 soft delete | `DELETE /books/{id}` | `Book::delete`, FK `RESTRICT` | `test_lending_rules_*`, `DatabaseInvariantsTest` |
| CA-04.1 / 04.2 paginated search | `GET /books` | `PgBookRepository::search` | `test_books_crud_and_search_hu_03_hu_04` |
| CA-04.3 404 when deleted | `GET /books/{id}` | `PgBookRepository::find` | `test_books_crud_and_search_hu_03_hu_04` |
| CA-05.1 / 05.2 / 05.3 members | `/members` | `MembersService` (no DELETE) | `test_members_hu_05` |
| CA-06.1 1 to 3 lines | `POST /loans` | `Loan::open` | `LoanTest`, `test_lending_rules_*` |
| CA-06.2 no repeated book | `POST /loans` | `Loan::open` | `test_lending_rules_ca_06_2_ca_06_4` |
| CA-06.3 decrement in the same operation | `POST /loans` | `Book::withdraw` + `PgLoanRepository::saveNew` | `test_lend_and_return_hu_06_hu_07`, `ConcurrencyTest` |
| CA-06.4 at most 3, with count | `POST /loans` | `Loan::open` → `MemberLimitExceeded` | `test_lending_rules_ca_06_2_ca_06_4` |
| CA-06.5 member with overdue loans | `POST /loans` | `Loan::open` → `MemberHasOverdue` | `test_overdue_blocks_and_lists_ca_06_5_hu_09` |
| CA-06.6 14 days and frozen values | `POST /loans` | `Loan::addItem` | `test_lend_and_return_hu_06_hu_07`, `LoanTest` |
| CA-07.1 / 07.2 return per line, once | `POST /loans/{id}/items/{item}/return` | `LoanItem::markReturned`, `PgLoanRepository::markReturned` | `test_lend_and_return_hu_06_hu_07` |
| CA-07.3 available ≤ total | — | `ck_book_available_range` | `ConcurrencyTest::test_database_check_is_the_last_barrier` |
| CA-07.4 / 08.1 / 08.3 computed status | `GET /loans/{id}` | `Loan::status` | `LoanTest::test_status_is_calculated_*` |
| CA-08.2 list with filters | `GET /loans` | `PgLoanRepository::search` (H-10) | `test_overdue_blocks_and_lists_*` |
| CA-09.1 / 09.2 overdue | `GET /loans/overdue` | `PgLoanRepository::overdue` | `test_overdue_blocks_and_lists_ca_06_5_hu_09` |
| CA-10.1 / 10.4 frozen report | `GET /reports/most-borrowed` | `PgReportReader` (ADR-004) | `test_report_is_frozen_and_admin_only_hu_10` |
| CA-10.2 inverted / empty range | `GET /reports/most-borrowed` | `DateRange` | `test_report_is_frozen_*`, `ValueObjectsTest` |
| CA-10.3 / 10.5 no members, admin only | `GET /reports/most-borrowed` | `PgReportReader`, `role:admin` | `test_report_is_frozen_*`, `test_token_and_role_*` |

## Frontend (`app`)

| Story | Screen |
|---|---|
| HU-01 | `screens/Login.jsx` |
| HU-02 | `screens/Librarians.jsx` |
| HU-03 / HU-04 | `screens/Catalog.jsx` |
| HU-05 | `screens/Members.jsx` |
| HU-06 | `screens/NewLoan.jsx` |
| HU-07 / HU-08 | `screens/Loans.jsx` |
| HU-09 | `screens/Overdue.jsx` |
| HU-10 | `screens/Report.jsx` |
