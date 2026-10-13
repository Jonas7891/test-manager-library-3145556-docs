# Decisiones sobre huecos y ambigüedades del spec

Cada fila es un punto donde el spec calla, se contradice o deja dos lecturas. La decisión **no amplía el alcance**: solo elige
una lectura y la deja escrita. Si una decisión es de arquitectura, tiene su ADR.

| # | Hueco / ambigüedad | Dónde | Decisión |
|---|---|---|---|
| H-01 | ADR-002 y D-04 hablan de un «testigo de versión por libro», pero `data-model.md` §3 no tiene columna `version`, y ese documento manda | ADR-002 vs data-model §3 | `available_copies` actúa como testigo (compare-and-set). No se añade columna. [ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md) |
| H-02 | T-01 y CA-01.3 piden `GET /health`, pero el contrato fija la base `/api/v1` | tasks / contrato §1 | Se exponen `/api/v1/health` y `/health`, con la misma respuesta |
| H-03 | El contrato §3.4 menciona `GET /books/{id}/cover` en el texto, pero no en la tabla de endpoints | contrato §3.4 | Se implementa: autenticado, devuelve el binario o 404 |
| H-04 | `GET /loans/overdue` choca con `GET /loans/{id}` | contrato §3.6 | `overdue` se declara antes y `{id}` solo acepta UUID |
| H-05 | T-21 pide 3 préstamos vencidos creados **por la API**, pero la API fecha cada préstamo con su propio reloj | T-21 vs HU-06 | El reloj del sistema admite un desplazamiento `CLOCK_OFFSET_DAYS` (0 por defecto). [ADR-006](adr/adr-006-reloj-desplazable-para-el-sembrador.md) |
| H-06 | CA-06.4 pide «422 con el conteo actual», pero no dice dónde va el conteo | spec / contrato | En `message` y en `details: [{field: "member_id", issue: "active_loans=N"}]` |
| H-07 | Laravel responde con su propio formato en 401, 404 y 422 | RNF-4 | Un único renderizador de errores convierte toda excepción al cuerpo `{code, message, details}` |
| H-08 | ¿Quién crea el esquema `library`? | ADR-001 | La primera migración (`CREATE SCHEMA IF NOT EXISTS library`); `infra` solo crea motor, usuario y volumen |
| H-09 | `PUT /books/{id}` no incluye el ISBN | contrato §3.4 | El ISBN no se puede cambiar después de crear el libro |
| H-10 | El filtro `status` de `GET /loans` usa un estado que no se almacena | HU-08 | Se calcula en SQL con `EXISTS` sobre `loan_item` (`returned_at IS NULL`, `due_at < ahora`) |
| H-11 | `from` y `to` del reporte son fechas «ambos inclusive», pero los instantes son UTC | contrato §3.7 | `loaned_at >= from 00:00Z` y `loaned_at < (to + 1 día) 00:00Z` |
| H-12 | Un libro dado de baja debe seguir viéndose en los préstamos pasados | ADR-003 | Las líneas se leen con sus valores congelados; nunca se unen al catálogo vivo |
| H-13 | `GET /loans/overdue` es un listado: ¿lleva paginación? | contrato §3.6 | Responde `{items, page, size, total}` como el resto de listados |
| H-14 | `GET /members` acepta `q`, pero no dice sobre qué campos busca | CA-05.2 | Coincidencia parcial sin distinguir mayúsculas en `full_name` o `document_number` |
| H-15 | ¿Se pueden reducir los ejemplares por debajo de los prestados? | CA-03.3 | No: la diferencia se aplica a los disponibles, y si quedan < 0 → 422 `VALIDATION_ERROR` |
| H-16 | El contrato §1 manda 401 sin token o con token vencido, pero el catálogo §2 no tiene un código para ese caso | contrato §1 / §2 | Se usa `INVALID_CREDENTIALS`, el único código 401 del catálogo, con el mensaje «Se requiere un token válido y vigente.». No se inventan códigos |
| H-17 | `PUT /books/{id}/cover` es `multipart/form-data`, pero PHP no interpreta multipart en un `PUT` real | contrato §3.4 | La ruta se mantiene; el cliente envía `POST` con `_method=PUT` (suplantación de método de Laravel). Documentado en el README de `api` |
| H-18 | `artisan serve` filtra las variables de entorno de sus procesos y la API terminaba usando SQLite | RNF-5 | El contenedor sirve con el servidor embebido de PHP, que hereda el entorno; `pgsql` también es la conexión por defecto |
