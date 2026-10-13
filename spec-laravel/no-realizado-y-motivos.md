# Lo que no se pudo realizar tal como dice el spec, y por qué

Al aplicar el spec (constitución, ADR, plan, tareas y contrato) aparecieron puntos que **no se pudieron cumplir al pie de la letra**:
el documento se contradice, el stack del reto no lo permite o la evidencia depende de algo que no existe en esta entrega. Cada
caso indica **qué dice el spec**, **qué lo impidió** y **qué se hizo en su lugar**. Ninguno se ocultó cambiando el spec original:
`spec-python/` y `spec-.net/` siguen intactos.

## 1. Errores o contradicciones del propio spec

| # | Qué dice el spec | Problema encontrado | Qué se hizo |
|---|---|---|---|
| E-1 | ADR-002 y D-04: control optimista con un **«testigo de versión» por libro** | `data-model.md` §3, que declara que gana sobre cualquier otro documento, **no tiene columna `version`**. Cumplir el ADR obligaba a cambiar el modelo sin que un requisito lo pidiera | **No se realizó como dice el ADR.** El testigo es el valor leído de `available_copies` (compare-and-set). Ver [ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md). Probado en `ConcurrencyTest` |
| E-2 | ADR-002: ante conflicto, 409 «**o reintenta una vez** si el ejemplar sigue disponible» | Es una alternativa abierta («o») que el contrato no refleja; reintentar ocultaría al bibliotecario que otra persona cambió el libro | **No se implementó el reintento.** Siempre responde 409 `BOOK_NOT_AVAILABLE`, la primera opción del ADR |
| E-3 | T-21: el sembrador crea **3 préstamos vencidos usando la API** | Imposible tal cual: `loaned_at` lo pone el reloj del servidor (HU-06) y el contrato no acepta fechas. Un préstamo creado hoy no vence antes de 14 días | Reloj desplazable solo para la demostración (`CLOCK_OFFSET_DAYS`, 0 por defecto). Ver [ADR-006](adr/adr-006-reloj-desplazable-para-el-sembrador.md) |
| E-4 | Contrato §1: 401 sin token o con token vencido | El catálogo de códigos §2 **no tiene código** para ese caso (`INVALID_CREDENTIALS` es para credenciales) | Se reutiliza `INVALID_CREDENTIALS` con otro mensaje, en lugar de inventar un código fuera del contrato (H-16) |
| E-5 | T-01 y CA-01.3: `GET /health`; contrato §1: base `/api/v1` | Las dos rutas no coinciden | Se exponen ambas (H-02) |
| E-6 | Contrato §3.4: `GET /books/{id}/cover` aparece en el texto pero no en la tabla de endpoints | Endpoint sin definición formal de rol | Se implementó para cualquier usuario autenticado (H-03) |
| E-7 | Contrato §3.6: `/loans/overdue` y `/loans/{id}` | La ruta `overdue` podía leerse como un id | `overdue` se declara antes y `{id}` solo acepta UUID (H-04) |

## 2. Lo que el stack del reto no permite hacer igual

| # | Qué dice el spec | Problema encontrado | Qué se hizo |
|---|---|---|---|
| S-1 | Art. I–III verificados con `import-linter` (Python) o `NetArchTest` (.NET) | Ninguna de las dos herramientas existe para PHP | Prueba propia `tests/Unit/Architecture/LayersTest.php`, que lee los `use` y falla ante una importación prohibida (T-19) |
| S-2 | Contrato §3.4: `PUT /books/{id}/cover` con `multipart/form-data` | **PHP no interpreta multipart en un `PUT` real**: el archivo llega vacío | Se conserva la ruta `PUT`; el cliente envía `POST` + `_method=PUT` (H-17) |
| S-3 | Laravel 11 (primera elección de esta traducción) | Composer **bloquea** todas las versiones de Laravel 11 por avisos de seguridad publicados | Se usó Laravel 13 sobre PHP 8.4 (D-01) |
| S-4 | RNF-5: un solo comando | `php artisan serve` filtra las variables de entorno y la API acababa usando SQLite | El contenedor sirve con el servidor embebido de PHP (H-18) |
| S-5 | Esquema `library` con solo las seis tablas | Laravel guarda su control de migraciones en una tabla `migrations` | Esa tabla vive en el esquema `public`, **fuera** de `library`: las consultas de data-model §10 muestran solo las seis tablas |

## 3. Lo que no se pudo realizar en esta entrega

| # | Qué dice el spec | Por qué no se pudo | Estado |
|---|---|---|---|
| N-1 | Art. IX: verificación con `gitleaks` o un `grep` **en CI** | Los repositorios no tienen CI y no estaba en el alcance crearlo | Se hizo a mano el 13-10-2026: ningún `.env` versionado y ningún secreto real en los seis repos. `phpunit.xml` solo tiene valores ficticios de prueba |
| N-2 | D-08: portadas en **S3 en producción** | No existe entorno de producción ni cuenta de S3 en la prueba | Solo existe el adaptador de disco. Gracias al puerto `CoverStorage`, S3 sería un adaptador nuevo más una línea en `PortBindingsServiceProvider` (OCP) |
| N-3 | T-22: «recorrido manual de punta a punta **en la feria**» | La evidencia es una demostración en vivo que ocurre después de la entrega | **Parcial**: el build compila y el recorrido está escrito en el README de `app`; la API que lo respalda está probada |
| N-4 | Archivos de bloqueo (`composer.lock`, `package-lock.json`) | La gobernanza del equipo limita cada commit a 400 líneas y esos archivos tienen miles | No se versionan; las dependencias se instalan al construir la imagen. Riesgo aceptado: las versiones exactas no quedan fijadas |
| N-5 | Commit inicial de cada repo | El commit base que entregó el instructor (`ariel5253`) trae un pie `Co-Authored-By` | No se reescribió porque es historia ajena y exigiría un `push --force`. Ningún commit de esta entrega lo lleva |
