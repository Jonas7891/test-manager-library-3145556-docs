# ADR-005 · El testigo de concurrencia es `available_copies`, sin columna `version`

- **Estado:** aceptada
- **Contexto:** [ADR-002](../../spec-python/adr/adr-002-concurrencia-optimista.md) pide control optimista con un «testigo de versión»
  por libro, pero [`data-model.md`](../../spec-python/data-model.md) §3, que es la fuente de verdad del modelo, no tiene ninguna columna
  `version`. Añadirla obligaría a cambiar el modelo de datos; ignorarla dejaría RNF-2 sin cubrir.
- **Decisión:** el propio valor leído de `available_copies` es el testigo. Al prestar, el adaptador ejecuta
  `UPDATE library.book SET available_copies = available_copies - 1 WHERE id = :id AND available_copies = :leido AND deleted_at IS NULL`.
  Si afecta 0 filas, otro proceso cambió el libro: se revierte la transacción completa y se responde **409 `BOOK_NOT_AVAILABLE`**.
  La última barrera sigue siendo `ck_book_available_range`.
- **Alternativas descartadas:** añadir `version integer` (cambia `data-model.md` sin que lo pida un requisito); `SELECT … FOR UPDATE`
  (ya descartado en ADR-002); confiar solo en el `CHECK` (la violación saldría como error 500 en vez de 409).
- **Consecuencias:** una edición concurrente del total de un libro también invalida un préstamo en curso: el préstamo falla con 409 y se
  reintenta. Es un caso raro y el resultado es seguro.
- **Cómo se verifica:** `tests/Feature/ConcurrencyTest.php` en `api`: dos préstamos simultáneos del último ejemplar; exactamente uno
  recibe 201.
