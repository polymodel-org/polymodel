# PolyModel Composition Strategy

## Decision

Favor composition over inheritance.

Composition should be a first-class feature of PolyModel.

Inheritance may be considered in the future but should not be part of the initial design.

## Motivation

Inheritance creates rigid hierarchies.

Example:

```text
Person
 └─ Employee
     └─ Contractor
```

These hierarchies become difficult to evolve and often do not map cleanly to:

- Relational databases
- Document databases
- DTO models
- Go structs
- TypeScript interfaces

## Composition Model

Instead of inheritance, reusable model parts are defined as components.

Example:

```hcl
component "Auditable" {

  field "createdAt" {
    type = timestamp
  }

  field "updatedAt" {
    type = timestamp
  }

}
```

```hcl
component "CurrencyAmount" {

  field "amount" {
    type = decimal
  }

  field "currency" {
    type = string
  }

}
```

## Using Components

```hcl
entity "Invoice" {

  use component.Auditable

  field "total" {
    use component.CurrencyAmount
  }

}
```

## Go Inspiration

PolyModel composition should feel similar to Go struct embedding.

```go
type Auditable struct {
    CreatedAt time.Time
    UpdatedAt time.Time
}

type CurrencyAmount struct {
    Amount   decimal.Decimal
    Currency string
}

type Invoice struct {
    Auditable
    Total CurrencyAmount
}
```

## Core Concepts

### Entity

Identity-bearing business object.

Examples:

- User
- Order
- Invoice

### Component

Reusable collection of fields.

Components do not have independent identity.

Examples:

- Auditable
- Address
- CurrencyAmount
- ContactInformation

### Relationship

Association between entities.

Examples:

- User -> Order
- Company -> Employee

## Advantages

### Reuse

Field groups can be reused consistently.

### Simpler Mappings

Composition maps naturally to:

- PostgreSQL
- SQLite
- Firestore
- MongoDB
- DTOs
- Go structs
- TypeScript interfaces

### Better Evolution

Components can evolve independently.

### Reduced Hierarchy Complexity

Avoids deep inheritance trees.

## Possible Future Syntax

```hcl
component "Address" {

  field "line1" {
    type = string
  }

  field "city" {
    type = string
  }

  field "country" {
    type = string
  }

}
```

```hcl
entity "Customer" {

  use component.Address

}
```

## Recommended Terminology

- Entity
- Field
- Component
- Relationship
- Projection
- Mapping

Avoid:

- Base Class
- Derived Class
- Abstract Entity

## Long-Term Direction

PolyModel should follow Go's philosophy:

- Prefer composition
- Keep models simple
- Encourage reusable building blocks
- Avoid unnecessary inheritance hierarchies

This approach aligns well with PolyModel's goal of generating multiple storage, transport, and language representations from a single canonical model.
