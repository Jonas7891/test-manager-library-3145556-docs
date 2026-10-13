# Tareas — Manager Library (Laravel + React): estado real

Mismas tareas que [`../spec-python/tasks.md`](../spec-python/tasks.md). Una tarea está **hecha** solo si su evidencia existe y se
puede ejecutar (artículo XII). Estados posibles: **hecha**, **parcial** (falta parte de la evidencia) y **pendiente**.

Rutas relativas a cada repositorio. La suite de `api` se corre con
`docker compose -f docker-compose.test.yml run --rm --build tests` y da **59 pruebas, 387 aserciones, todas en verde**
(13-10-2026).

| # | Tarea | Estado | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto y `GET /health` | hecha | `api`: `AuthAndCatalogContractTest::test_health_is_public_t_01`; `curl localhost:8000/api/v1/health` → `{"status":"ok"}` |
| T-02 | Migración inicial con semilla | hecha | `api`: `database/migrations/2026_10_13_000002_*`; consultas de data-model §10 en [`evidencia/t-02-esquema.txt`](evidencia/t-02-esquema.txt) (ninguna columna con `DEFAULT`); `DatabaseInvariantsTest::test_no_column_has_a_default_and_there_is_no_status_column` |
| T-03 | Objetos de valor | hecha | `api`: `tests/Unit/Domain/ValueObjectsTest.php` |
| T-04 | `Book` | hecha | `api`: `tests/Unit/Domain/BookTest.php` (nunca < 0, nunca > total) |
| T-05 | `Loan` | hecha | `api`: `tests/Unit/Domain/LoanTest.php`, con reloj fijo: `open`, `overdue`, `closed` |
| T-06 | `Member` y `User` | hecha | `api`: `MemberAndUserTest` (`"Ana "` → `ana`; rol `admin` rechazado) |
| T-07 | Autenticación con fakes | hecha | `api`: `UseCasesTest::test_login_*`, `test_librarian_creation_rules_hu_02` |
| T-08 | Libros con fakes | hecha | `api`: `UseCasesTest::test_isbn_unique_*`, `test_cannot_delete_book_with_open_loans_ca_03_5` |
| T-09 | Prestar con fakes | hecha | `api`: `UseCasesTest::test_lend_*`, `test_member_with_overdue_is_blocked_ca_06_5` |
| T-10 | Devolver y vencidos | hecha | `api`: `UseCasesTest::test_return_restocks_once_and_twice_fails_hu_07`; `LoansContractTest::test_overdue_blocks_and_lists_*` |
| T-11 | Reporte de más prestados | hecha | `api`: `UseCasesTest::test_report_keeps_frozen_title_after_rename_adr_004`; `LoansContractTest::test_report_is_frozen_and_admin_only_hu_10` |
| T-12 | Repositorios de libro, socio y usuario | hecha | `api`: `tests/Feature/*ContractTest` contra PostgreSQL real |
| T-13 | Repositorio de préstamos + testigo | hecha | `api`: `tests/Feature/ConcurrencyTest.php`: del último ejemplar se registra un solo préstamo ([ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md)) |
| T-14 | Hash y JWT | hecha | `api`: el contrato comprueba que ninguna respuesta incluye `password`; Argon2id + JWT HS256 de 8 h (`expires_in: 28800`) |
| T-15 | Portadas en disco | parcial | `api`: `test_cover_upload_validates_type_and_size_ca_03_4` (PNG aceptado, tipo falso → 422, sin portada → `null`). El rechazo de un archivo de 3 MB está en `BookCoverService` pero no tiene prueba propia |
| T-16 | Adaptador HTTP y cuerpo de error único | hecha | `api`: `AuthAndCatalogContractTest` y `LoansContractTest` recorren las rutas y los códigos de §2 |
| T-17 | Composición y admin inicial | hecha | `api`: `PortBindingsServiceProvider`; `library:bootstrap-admin` al arrancar; login de admin con la base recién creada |
| T-18 | `ck_book_total_positive`, `ck_user_role` | hecha | `api`: migración `2026_10_13_000003_*`; `DatabaseInvariantsTest`; aparecen en [`evidencia/t-02-esquema.txt`](evidencia/t-02-esquema.txt). En esta traducción, data-model §4 las marca ahora como **motor** |
| T-19 | Pruebas de arquitectura | hecha | `api`: `tests/Unit/Architecture/LayersTest.php` (incluye una prueba de que el verificador detecta importaciones prohibidas) |
| T-20 | `infra`: `docker compose up` | hecha | `infra`: `docker compose up -d --build` levanta db, api y app; el repo no contiene `CREATE TABLE` |
| T-21 | `tool`: sembrador por la API | hecha | `tool`: `node seed/seed.mjs --check` → «Vencidos: 3» y 12 filas en el reporte (salida en el README de `tool`) |
| T-22 | `app`: recorrido de punta a punta | parcial | `app`: `npm run build` sin errores y el recorrido escrito en el README. Falta grabar el recorrido manual (capturas) en la feria |
| T-23 | `page`: sitio estático | hecha | `page`: HTML y CSS sin JavaScript (`grep -c "fetch\|<script" index.html` → 0) y diseño responsivo |

## Requisitos no funcionales

| # | Estado | Evidencia |
|---|---|---|
| RNF-1 (búsqueda < 300 ms con 5 000 libros) | pendiente | No hay prueba de carga; existe el índice `ix_book_category_title_active` |
| RNF-2 (último ejemplar) | hecho | `ConcurrencyTest` |
| RNF-3 (contraseña nunca en claro ni en logs) | hecho | Argon2id; `assertErrorBody` comprueba que ninguna respuesta incluye `password`; los errores de dominio no se registran en el log |
| RNF-4 (cuerpo de error único) | hecho | `ErrorRenderer`; todas las pruebas de contrato usan `assertErrorBody` |
| RNF-5 (un comando, base vacía) | hecho | README de `infra` |
