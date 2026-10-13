# Arquitectura — Manager Library (Laravel)

La misma arquitectura **hexagonal** de [`../spec-python/architecture.md`](../spec-python/architecture.md), expresada con las
convenciones de Laravel. Las reglas de dependencia (artículos I a III) no cambian.

## 1. Capas y dependencias

```mermaid
flowchart LR
    subgraph Entrada
      HTTP["app/Adapters/Inbound/Http<br/>(controladores Laravel)"]
    end
    subgraph Núcleo
      APP["app/Application<br/>casos de uso + puertos"]
      DOM["app/Domain<br/>PHP puro"]
    end
    subgraph Salida
      DB["app/Adapters/Outbound/Persistence<br/>(Query Builder + migraciones)"]
      STO["app/Adapters/Outbound/Storage<br/>portadas"]
      SEC["app/Adapters/Outbound/Security<br/>Argon2id y JWT"]
    end
    HTTP --> APP --> DOM
    DB -. implementa puertos de .-> APP
    STO -. implementa .-> APP
    SEC -. implementa .-> APP
```

## 2. Estructura de carpetas (repo `api`)

```text
app/
├── Domain/              # Book, Loan, LoanItem, Member, User, LoanStatus; ValueObjects/; Exceptions/
├── Application/
│   ├── Ports/           # BookRepository, LoanRepository, MemberRepository, UserRepository,
│   │                    # ReportReader, Clock, PasswordHasher, TokenIssuer, CoverStorage
│   └── Services/        # un caso de uso por clase (LendBooksService, ReturnItemService, …)
├── Adapters/
│   ├── Inbound/Http/    # Controllers/, Middleware/ (JwtAuth, RequireRole), ErrorRenderer, Paginated
│   └── Outbound/        # Persistence/, Storage/, Security/, Clock/
├── Bootstrap/           # PortBindingsServiceProvider.php — ÚNICO punto de composición (Art. III)
└── Console/             # BootstrapAdminCommand (admin inicial desde entorno)
database/migrations/     # todo el DDL (Art. V, ADR-001)
routes/api.php           # rutas de api-contract.md §3
```

## 3. Equivalencias con Laravel

| Concepto del spec | En Laravel |
|---|---|
| Puerto | `interface` en `app/Application/Ports` |
| Adaptador | Clase en `app/Adapters/Outbound/*` que implementa el puerto |
| Composición | `PortBindingsServiceProvider::register()` con `$this->app->singleton(Port::class, Adapter::class)` |
| Filtro global de baja lógica | Consulta base `activeBooks()` en `PgBookRepository` (`deleted_at IS NULL`) |
| Transacción de préstamo | `DB::transaction()` dentro del adaptador `PgLoanRepository::saveNew()` |
| Errores de dominio → HTTP | `ErrorRenderer` registrado en `bootstrap/app.php` (`withExceptions`) |
| Autorización por rol | Middleware `role:admin,librarian` |

**Eloquent no se usa**: sus convenciones (`created_at`, `updated_at`, ids autoincrementales) contradicen `data-model.md` §3 y
acoplarían el dominio al ORM. El Query Builder vive solo en los adaptadores.

## 4. Flujo: registrar un préstamo

```mermaid
sequenceDiagram
    participant C as Cliente
    participant H as LoanController
    participant S as LendBooksService
    participant L as PgLoanRepository
    C->>H: POST /api/v1/loans
    H->>S: lend(memberId, bookIds, userId)
    S->>L: líneas sin devolver del socio (Q7)
    S->>S: ¿vencidos? 409 · ¿supera 3? 422
    S->>S: Loan::open → Book::withdraw por cada libro
    S->>L: saveNew(loan, books) en una transacción
    L->>L: UPDATE book … WHERE available_copies = leído (ADR-005)
    L-->>S: ok o BookNotAvailable
    S-->>H: préstamo
    H-->>C: 201 o 409
```

## 5. Qué no hay

Sin eventos, colas, caché, Eloquent ni Sanctum. Igual que el spec: se añaden si un requisito lo pide.
