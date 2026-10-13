# ADR-006 · Desplazamiento del reloj solo para la demostración

- **Estado:** aceptada
- **Contexto:** T-21 pide que el sembrador cree préstamos **por la API** y que después `/loans/overdue` devuelva 3 líneas. Pero
  `loaned_at` lo pone el servidor con su reloj (HU-06) y el contrato no admite enviar una fecha. Sin otra salida, ningún préstamo
  creado hoy vence antes de 14 días.
- **Decisión:** el adaptador `SystemClock` suma `CLOCK_OFFSET_DAYS` días a la hora real. El valor por defecto es `0`, no aparece en el
  contrato HTTP y solo se cambia al levantar la API para la fase «pasada» del sembrador (por ejemplo, `-20`). El dominio no se
  entera: ya recibe la hora por el puerto `Clock` (D-05).
- **Alternativas descartadas:** insertar préstamos por SQL (lo prohíbe `data-model.md` §9); un campo `loaned_at` en `POST /loans`
  (amplía el contrato y abre un hueco de integridad); una cabecera para cambiar la hora por petición (cualquier cliente podría
  falsear fechas).
- **Consecuencias:** con un desplazamiento distinto de 0, todo el sistema vive en otra fecha; por eso solo se usa en demostración y
  nunca va en el `.env.example` con otro valor que `0`.
- **Cómo se verifica:** el README de `tool` muestra las dos fases. Después de correrlas, `GET /loans/overdue` devuelve 3 líneas.
