# PolyModel

**Define data models once. Generate storage schemas, transport contracts, and language types everywhere.**

🌐 **Website:** https://polymodel.org

PolyModel is an open specification for defining canonical data models independent of databases, APIs, programming languages, or storage technologies.

A single PolyModel definition can be projected into:

- PostgreSQL
- SQLite
- Oracle
- Firestore
- MongoDB
- OpenAPI
- GraphQL
- JSON Schema
- Go structs
- TypeScript interfaces
- C# classes
- and more

PolyModel provides a single source of truth for your application's data model.

---

## Why PolyModel?

Today most teams define the same model multiple times:

```text
Database Schema
        ↓
ORM Models
        ↓
DTO Objects
        ↓
API Contracts
        ↓
Frontend Types
```

Every representation eventually drifts.

PolyModel aims to solve this problem by defining the canonical model once and generating all target representations from it.

```text
PolyModel
     ↓
 PostgreSQL
 Firestore
 OpenAPI
 GraphQL
 Go
 TypeScript
```

---

# Key Differentiator

Most existing technologies focus on a single layer.

| Project | Primary Focus |
|----------|----------|
| TypeSpec | API contracts |
| OpenAPI | HTTP APIs |
| Prisma | Database schemas |
| DBML | Database modelling |
| Atlas | Database schemas & migrations |
| PolyModel | Canonical data models |

PolyModel introduces a separation between:

```text
Entity Model
      ↓
Storage Model
      ↓
Transport Model
      ↓
Language Model
```

This allows the same logical model to be represented differently for different targets.

For example:

```text
User
 └─ Orders
     └─ OrderItems
```

can become:

```text
Firestore

users/{userId}
  orders/{orderId}
    items/{itemId}
```

or:

```text
PostgreSQL

users
orders
order_items
```

without changing the source model.

---

# Core Concepts

PolyModel v0 has three structural concepts — **Entity**, **Collection**, and
**Recordset** — plus reusable **Components**. Each concept has its own attribute type,
named for what it is: `property` (Entity), `field` (Collection), `column` (Recordset).
The authored format is **HCL**.

> The authoritative, machine-checkable specification lives in
> [`spec/features/core-model`](spec/features/core-model/README.md), with the rationale
> recorded as decisions in [`spec/decisions`](spec/decisions/README.md). A reference Go
> implementation is at [`polymodel-go`](https://github.com/polymodel-org/polymodel-go).

## Entity

A logical, identity-bearing business object — the semantic anchor. An Entity has
**properties** (its canonical attributes), may embed **components**, and may declare a
**key**.

```hcl
entity "User" {
  key = ["id"]

  property "id" {
    type = "uuid"
  }

  property "email" {
    type   = "string"
    unique = true
  }
}
```

A property may **associate** to another Entity (a relationship) or **reuse** another
property's definition:

```hcl
entity "Order" {
  key = ["id"]

  property "id" {
    type = "uuid"
  }

  property "user" {
    entity = "User" # association to another Entity
  }
}
```

---

## Collection

A named, queryable **data source** that holds records — the thing you read `FROM`.
A Collection has a `kind`: `editable` (a stored table or file-system collection) or
`computed` (a view). There is no separate Table or View type — the distinction is the
kind. A Collection's attributes are **fields**: loose and schemaless-capable, they may
**bind** to an Entity property and may declare an enforced **foreign key**.

```hcl
collection "users" {
  kind = "editable"

  field "id" {
    type = "uuid"
    bind = "User.id"
  }
}

collection "active_users" {
  kind  = "computed"
  query = "from users where active" # held behind an opaque seam (DTQL planned)
}
```

---

## Recordset

The tabular **shape of a result** — a query or stored-procedure output. A Recordset has
ordered **columns** (strict and tabular, unlike loose collection fields) and may carry
a query.

```hcl
recordset "user_summary" {
  key = ["userId"]

  column "userId" {
    type = "uuid"
    bind = "User.id"
  }

  column "orderCount" {
    type   = "int"
    source = "count(orders)" # produced-by expression
  }
}
```

---

## Component

A reusable, named group of fields, embedded into Entities — favoring composition over
inheritance.

```hcl
component "Auditable" {
  field "createdAt" { type = "datetime" }
  field "updatedAt" { type = "datetime" }
}

entity "Invoice" {
  use = ["Auditable"]

  property "total" {
    type = "decimal"
  }
}
```

---

## Storage Projection

How entities are represented in a storage system.

### PostgreSQL

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP NOT NULL
);
```

### Firestore

```text
users/{userId}
  orders/{orderId}
```

---

## Transport Projection

How entities are represented when transported across process boundaries.

### OpenAPI

```yaml
User:
  type: object
  properties:
    id:
      type: string
    email:
      type: string
```

### GraphQL

```graphql
type User {
  id: ID!
  email: String!
}
```

---

## Language Projection

### Go

```go
type User struct {
    ID    string
    Email string
}
```

### TypeScript

```ts
export interface User {
  id: string;
  email: string;
}
```

---

# Hierarchical Data Example

PolyModel treats hierarchy as a logical concern rather than a storage concern.

## Canonical Model

```hcl
entity "User" {
  key = ["id"]

  property "id"    { type = "uuid" }
  property "email" { type = "string" }
}

entity "Order" {
  key = ["id"]

  property "id"        { type = "uuid" }
  property "user"      { entity = "User" } # association
  property "createdAt" { type = "timestamp" }
}

entity "OrderItem" {
  key = ["id"]

  property "id"       { type = "uuid" }
  property "order"    { entity = "Order" } # association
  property "quantity" { type = "int" }
}
```

## Firestore Projection

```text
users/{userId}
  orders/{orderId}
    items/{itemId}
```

## PostgreSQL Projection

```text
users
orders
order_items
```

with generated foreign keys:

```text
orders.user_id → users.id

order_items.order_id → orders.id
```

## DTO Projection

```ts
interface UserDto {
  id: string;
  email: string;
  orders: OrderDto[];
}

interface OrderDto {
  id: string;
  createdAt: string;
  items: OrderItemDto[];
}
```

---

# Migrations

> **Deferred — not part of v0.** Migrations are out of scope for the core model and
> will be specified separately. The sketch below is illustrative of the future
> direction, not the current spec.

PolyModel aims to support schema evolution through versioned migrations.

```hcl
migration "v2" {

  entity "User" {

    property "displayName" {
      type     = "string"
      optional = true
    }

  }

}
```

Generators may produce:

- PostgreSQL migrations
- SQLite migrations
- Oracle migrations
- Firestore index updates
- MongoDB migrations

---

# Design Goals

- Human-readable
- Version-controllable
- Storage-agnostic
- Transport-agnostic
- Language-agnostic
- Open specification
- Extensible
- Generator-friendly
- Suitable for AI-assisted development

---

# Non-Goals

PolyModel is not:

- An ORM
- A database
- An API framework
- A migration engine
- A code generator implementation

PolyModel defines the specification. Implementations build on top of it.

---

# Relationship to TypeSpec

PolyModel and TypeSpec are complementary.

```text
PolyModel
    ↓
TypeSpec
    ↓
OpenAPI
```

PolyModel focuses on canonical data models.

TypeSpec focuses on API contracts.

A future PolyModel generator may produce TypeSpec definitions directly.

---

# Roadmap

## v0.1 — core model ✅

- Entity / Collection / Recordset concepts
- `property` / `field` / `column` attribute types
- `Collection.Kind` (editable | computed) — no separate Table/View type
- Components (composition; Entities only)
- Opaque query seam (DTQL intended)
- HCL syntax + type system + constraints + located-error validation
- SpecScore specification (`spec/`) and a reference Go implementation (`polymodel-go`)

## v0.2

- PostgreSQL generator
- SQLite generator
- Go generator
- TypeScript generator

## v0.3

- Firestore projection
- MongoDB projection
- JSON Schema generator
- OpenAPI generator

## v0.4

- Migration specification
- Index definitions
- Constraints
- Validation rules

## v0.5

- GraphQL generator
- TypeSpec generator
- VS Code extension
- Formatter

## v1.0

- Stable specification
- Compliance suite
- Reference implementations
- Community governance

---

# Status

The **v0 core model is defined and implemented** — see
[`spec/features/core-model`](spec/features/core-model/README.md) for the specification
and [`polymodel-go`](https://github.com/polymodel-org/polymodel-go) for the reference
implementation. Generators (PostgreSQL, Firestore, inGitDB, language types, …) and a
serialization for non-Go consumers are still ahead.

The specification is early and may change before v1.0. Feedback, ideas, and
contributions are welcome.

---

# License

Apache License 2.0
