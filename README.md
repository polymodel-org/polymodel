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

## Entity

Logical business object.

```hcl
entity "User" {
  field "id" {
    type = uuid
  }

  field "email" {
    type = string
    unique = true
  }
}
```

---

## Relationship

Logical relationship between entities.

```hcl
entity "Order" {
  parent = entity.User

  field "id" {
    type = uuid
  }

  field "createdAt" {
    type = timestamp
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
  field "id" {
    type = uuid
  }

  field "email" {
    type = string
  }
}

entity "Order" {
  parent = entity.User

  field "id" {
    type = uuid
  }

  field "createdAt" {
    type = timestamp
  }
}

entity "OrderItem" {
  parent = entity.Order

  field "id" {
    type = uuid
  }

  field "quantity" {
    type = int
  }
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

PolyModel supports schema evolution through versioned migrations.

```hcl
migration "v2" {

  entity "User" {

    field "displayName" {
      type = string
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

## v0.1

- Entity definitions
- Fields
- Relationships
- Hierarchies
- HCL-based syntax
- Specification document
- Example models

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

PolyModel is currently in the design phase.

The specification is evolving and may change significantly before v1.0.

Feedback, ideas, and contributions are welcome.

---

# License

Apache License 2.0
