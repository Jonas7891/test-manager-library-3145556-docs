# ADR-007 · JWT con `firebase/php-jwt` en lugar de Sanctum

- **Estado:** aceptada
- **Contexto:** el contrato (§4) y D-11 piden un **JWT firmado HS256**, con vigencia de 8 horas y los claims `sub` y `role`, sin estado
  en el servidor. La opción por defecto de Laravel, Sanctum, emite tokens opacos que se guardan en una tabla
  `personal_access_tokens`, una tabla que no existe en `data-model.md`.
- **Decisión:** el puerto `TokenIssuer` lo implementa `JwtTokenIssuer` con `firebase/php-jwt` y la clave `JWT_SECRET`. El middleware
  `JwtAuth` valida firma y expiración, y `RequireRole` comprueba el rol del claim.
- **Alternativas descartadas:** Sanctum (añade una tabla y el token no es JWT); `tymon/jwt-auth` (paquete grande, acoplado a Eloquent).
- **Consecuencias:** no se puede revocar un token antes de que caduque; lo mismo ocurre en el spec original, porque el JWT no guarda
  estado.
- **Cómo se verifica:** las pruebas de contrato comprueban que sin token se recibe 401, que con un token de `librarian` en
  `/reports/most-borrowed` se recibe 403, y que `expires_in` es 28800.
