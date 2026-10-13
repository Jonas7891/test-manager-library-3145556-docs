# Plan técnico — Manager Library (Laravel + React)

Traducción de [`../spec-python/plan.md`](../spec-python/plan.md). El negocio no cambia; cambia la tecnología.

## 1. Decisiones técnicas

| # | Decisión | Elección (reto) | Por qué |
|---|---|---|---|
| D-01 | Lenguaje y runtime | PHP 8.3 + Laravel 11 | Stack del reto |
| D-02 | Framework HTTP | Rutas de Laravel con prefijo `api/v1`, controladores delgados | Contrato en `api-contract.md` |
| D-03 | Persistencia | Migraciones de Laravel (SQL explícito) + Query Builder sobre PostgreSQL 16, esquema `library` | ADR-001: el esquema es de las migraciones |
| D-04 | Concurrencia | Compare-and-set sobre `available_copies` + `CHECK ck_book_available_range` ([ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md)) | RNF-2 sin añadir columnas al modelo |
| D-05 | Fechas | `timestamptz`, aplicación en UTC; el reloj es el puerto `Clock` | Probar vencimientos sin esperar 14 días |
| D-06 | Reporte | Una consulta agregada detrás del puerto `ReportReader` | ADR-004 |
| D-07 | Objetos de valor | Clases PHP `final readonly`: `Isbn`, `Quantity`, `DocumentNumber`, `DateRange` | Sin tabla, sin identidad |
| D-08 | Portadas | Disco local (`COVERS_DIR`) detrás de `CoverStorage`; la base guarda solo `cover_key` | Clave opaca |
| D-09 | Contraseñas | Argon2id (`password_hash`) detrás de `PasswordHasher` | Artículo IX |
| D-10 | Arranque | Comando `php artisan library:bootstrap-admin` desde `ADMIN_USERNAME` / `ADMIN_PASSWORD` | La base no puede producir el hash |
| D-11 | Autenticación | JWT HS256, 8 h, `firebase/php-jwt`, clave `JWT_SECRET` ([ADR-007](adr/adr-007-jwt-propio-en-vez-de-sanctum.md)) | Lo pide el contrato §4 |
| D-12 | Pruebas | PHPUnit: dominio puro, aplicación con fakes, contrato contra PostgreSQL real | Pirámide de pruebas |

## 2. Configuración por entorno

| Variable | Obligatoria | Ejemplo |
|---|---|---|
| `DB_HOST` / `DB_PORT` / `DB_DATABASE` / `DB_USERNAME` / `DB_PASSWORD` | sí | `db` / `5432` / `library` / `library` / *(en `.env`)* |
| `JWT_SECRET` | sí | cadena larga aleatoria |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | sí | definidos en `.env`, nunca en el repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | sí | `http://localhost:5173` |
| `CLOCK_OFFSET_DAYS` | no | `0` (ver [ADR-006](adr/adr-006-reloj-desplazable-para-el-sembrador.md)) |

## 3. Estrategia de pruebas

| Nivel | Qué prueba | Dónde (repo `api`) |
|---|---|---|
| Dominio | Invariantes: máximo 3, vencimiento a 14 días, disponibles en rango | `tests/Unit/Domain/` |
| Aplicación | Casos de uso con fakes de los puertos | `tests/Unit/Application/` |
| Contrato | Rutas, códigos de §2, cuerpo de error único | `tests/Feature/` (PostgreSQL real) |
| Arquitectura | Artículos I, II y III | `tests/Unit/Architecture/` |
| Concurrencia | Dos préstamos del último ejemplar | `tests/Feature/ConcurrencyTest.php` |

## 4. Orden de construcción

Igual que el spec: esqueleto y salud → migración inicial con semilla → dominio → casos de uso con fakes → adaptadores → API →
pruebas de contrato y concurrencia → infra, tool, app y page.
