# spec-laravel — Manager Library traducido a React + Laravel

Esta carpeta **no reescribe** el spec. `spec-python/` y `spec-.net/` siguen siendo la fuente de verdad del negocio
(historias, criterios de aceptación, reglas, contrato HTTP y modelo de datos). Aquí se documenta **cómo se trasladó**
ese spec al stack del reto y con qué evidencia se da por hecha cada tarea.

## Cómo se leyó el spec

1. `constitution.md` → los 12 artículos se mantienen; solo cambia la herramienta con que se verifican.
2. `spec.md`, `data-model.md`, `api-contract.md` → se respetan **sin cambios** (independientes del lenguaje).
3. `plan.md` / `architecture.md` → se traducen a Laravel en [`plan.md`](plan.md) y [`architecture.md`](architecture.md).
4. `tasks.md` → se reutilizan T-01…T-23 con su evidencia; el estado real está en [`tasks.md`](tasks.md).
5. Donde el spec es ambiguo o se contradice, la decisión tomada está en [`decisiones-y-huecos.md`](decisiones-y-huecos.md)
   y, si es de arquitectura, en un ADR nuevo (`adr/adr-005` en adelante).

`spec-python/` y `spec-.net/` solo difieren en framework, ORM, algoritmo de hash, herramienta de pruebas de arquitectura y
rutas de carpetas; el negocio es idéntico. Se tomó `spec-python/` como base de lectura.

## Qué cambia frente a Python / .NET

| Tema | Python | .NET | Reto (este documento) |
|---|---|---|---|
| Backend | FastAPI | ASP.NET Core | **Laravel 13 (PHP 8.4)** |
| Persistencia | SQLAlchemy + Alembic | EF Core + migraciones | **Migraciones de Laravel + Query Builder** |
| Hash | Argon2id | BCrypt | **Argon2id** (`password_hash`) |
| Token | JWT HS256 | JWT HS256 | **JWT HS256** con `firebase/php-jwt` ([ADR-007](adr/adr-007-jwt-propio-en-vez-de-sanctum.md)) |
| Pruebas | pytest | xUnit | **PHPUnit** |
| Arquitectura | import-linter | NetArchTest | **Test PHPUnit que inspecciona los `use`** (`tests/Unit/Architecture`) |
| Frontend | React | React | **React + Vite** |
| Motor | PostgreSQL 16 | PostgreSQL 16 | **PostgreSQL 16** (sin cambio) |

## Índice

| Archivo | Contenido |
|---|---|
| [`plan.md`](plan.md) | Decisiones técnicas D-01…D-12 traducidas |
| [`architecture.md`](architecture.md) | Capas hexagonales dentro de Laravel y carpetas reales |
| [`decisiones-y-huecos.md`](decisiones-y-huecos.md) | Ambigüedades del spec y la decisión tomada |
| [`trazabilidad.md`](trazabilidad.md) | HU / CA → endpoint → clase → prueba |
| [`tasks.md`](tasks.md) | Estado real de T-01…T-23 con enlace a la evidencia |
| [`no-realizado-y-motivos.md`](no-realizado-y-motivos.md) | Lo que no se pudo realizar tal como dice el spec y por qué |
| [`adr/`](adr/) | ADR-005, ADR-006 y ADR-007 |
| `evidencia/` | Salidas de comandos usadas como evidencia |

## Repositorios

| Repo | Contenido |
|---|---|
| `test-manager-library-3145556-api` | Backend Laravel |
| `test-manager-library-3145556-app` | Frontend React |
| `test-manager-library-3145556-infra` | `docker compose` con motor vacío |
| `test-manager-library-3145556-tool` | Sembrador de demostración por la API |
| `test-manager-library-3145556-page` | Sitio público estático |
