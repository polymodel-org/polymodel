---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: PolyModel Core Model

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model?op=explore) | [Edit](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model?op=edit) | [Ask question](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model?op=ask) | [Request change](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model?op=request-change) |
**Status:** Approved
**Source Ideas:** core-model

## Summary

The canonical PolyModel data model: Entity, Collection, and Recordset with their attribute types, decoupling semantic model from physical schema and query results.

## Contents

| Child | Description |
|---|---|
| [entity](entity/README.md) | Entity: the semantic, identity-bearing business object with properties, components, and a key. |
| [collection](collection/README.md) | Collection: a named, FROM-able data source (Editable table/FS-collection or Computed view) with loose, schemaless-capable fields. |
| [recordset](recordset/README.md) | Recordset: the tabular shape of a query or stored-procedure result, with strict columns. |

## Problem

Existing tools (notably DataTug) tangle three distinct concerns into one layer: the
**semantic model** (what an `Order` *is*), the **physical schema** (the table or view
that stores it), and **query results** (the columns a query or stored procedure
returns). Because they are entangled, a model cannot be reused across storage,
transport, and language targets — each target forces a redefinition that drifts.

PolyModel's core model decouples these into distinct first-class concepts so one
canonical model can project to many targets. This parent Feature specifies the
**foundations** shared across all three concepts — the concept set, the attribute-type
distinction, the binding model, the type system, Component reuse, the query seam,
authoring, and validation. The per-concept rules live in the sub-features:
[entity](entity/README.md), [collection](collection/README.md), and
[recordset](recordset/README.md).

The `polymodel-go` v0 library (in-memory model + HCL parsing + validation) is the
reference implementation; the acceptance criteria are written against its observable
parse/validate behavior. This Feature tree defines the **model only** — not generators,
serialization formats, catalog references, or migrations (see [Out of Scope](#out-of-scope)).

## Behavior

A PolyModel definition is a set of named objects of three structural concepts —
**Entity**, **Collection**, **Recordset** — plus reusable **Component**s. Each concept
has its own attribute type, named for what that layer is: `property` (Entity),
`field` (Collection), `column` (Recordset). This parent specifies the rules common to
all three; each concept's own rules are in its sub-feature.

### Concepts

#### REQ: structural-concepts

A PolyModel model MUST support exactly three structural concepts — **Entity**,
**Collection**, and **Recordset** — and the reusable **Component**. Each is a named,
top-level object within a model. Names MUST be unique within their concept kind. No
other structural concept exists in v0.

#### REQ: attribute-types-distinct

The three concepts MUST use three distinct attribute types, not one shared type:
an Entity has **properties**, a Collection has **fields**, a Recordset has **columns**.
The terms are normative because the layers differ in character: a property is the
canonical semantic definition, a field is a loose schemaless-capable attribute, and a
column is a strict tabular attribute.

### Attributes and binding

#### REQ: binding-local-primary

When a Collection field or a Recordset column binds to an Entity property, the **local
attribute is primary**: the Collection owns its field and the Recordset owns its column,
and the binding is an optional reference to the property. The local attribute MAY
override or add concerns (storage, nullability, result expression). A field or column
with no binding is valid.

### Component

#### REQ: component-entities-only

A **Component** is a reusable, named group of fields with no independent identity. In
v0, Components MAY be embedded into **Entities only** — not into Collections or
Recordsets.

### Type system

#### REQ: type-system

A property/field/column type MUST be one of the v0 primitive set —
`string`, `int`, `float`, `bool`, `decimal`, `uuid`, `date`, `time`, `datetime`,
`document` (a.k.a. `json`), `any` — or a reference (Component embed, or Entity
association for properties). Constraints are limited to the v0 set: `required`,
`unique`, length (min/max), `pattern`, `enum`. A type or constraint outside these sets
is invalid in v0. Fuller reconciliation (localization, generic maps, additional
constraints) is deferred to a separate type-system decision.

### Authoring and consumption

#### REQ: hcl-authored

The authored format of a PolyModel model MUST be HCL, matching the specification.
Parsing HCL into the in-memory model is part of v0.

#### REQ: go-in-memory

Consumers MUST be able to use the model via the in-memory representation
(`polymodel-go`) directly, without a serialized interchange file. JSON/YAML emission is
out of scope for v0.

#### REQ: query-seam

The query body of a Computed Collection or a Recordset MUST be represented behind a
**placeholder seam** — an opaque query reference that the model carries without
interpreting. DTQL is the intended query AST; v0 MUST NOT depend on a concrete DTQL
implementation, so the seam allows a DTQL AST to be swapped in later without reshaping
the model.

### Validation

#### REQ: validation

A PolyModel model MUST be validatable, producing **located** errors (object + attribute
context) on violation. Validation MUST resolve intra-model references — property reuse,
Component embeds, Entity associations, Collection foreign keys, and field/column
property bindings — and MUST reject references whose target does not exist.

## Acceptance Criteria

### AC: model-concepts
**Requirements:** core-model#req:structural-concepts, core-model#req:attribute-types-distinct

**Scenario:** Three concepts with their own attribute types parse.
**Given** an HCL model declaring an Entity with properties, a Collection with fields, and a Recordset with columns
**When** the model is parsed
**Then** it loads with the three concepts, each exposing its own attribute type (property / field / column), and a model declaring an unknown top-level concept is rejected with a located error.

### AC: binding-precedence
**Requirements:** core-model#req:binding-local-primary

**Scenario:** The local attribute is primary over its property binding.
**Given** a Collection field that binds to an Entity property and overrides its nullability
**When** the model is validated
**Then** the local field is primary, the override applies, the property binding resolves, and a field with no binding at all is also valid.

### AC: component-scope
**Requirements:** core-model#req:component-entities-only

**Scenario:** Components embed into Entities only in v0.
**Given** a Component embedded into an Entity and the same Component embedded into a Collection
**When** the model is validated
**Then** the Entity embed validates and the Collection embed is rejected with a located error in v0.

### AC: type-system
**Requirements:** core-model#req:type-system

**Scenario:** Only v0 primitives and constraints are accepted.
**Given** properties, fields, and columns using v0 primitive types and v0 constraints
**When** the model is validated
**Then** they validate, and a type or a constraint outside the v0 sets is rejected with a located error.

### AC: query-seam-opaque
**Requirements:** core-model#req:query-seam

**Scenario:** A query body is retained without interpretation.
**Given** a model whose Computed Collection or Recordset carries a query
**When** the model is parsed
**Then** the query is retained as an opaque seam value the model does not interpret, and the model loads without any concrete DTQL implementation present.

### AC: authoring
**Requirements:** core-model#req:hcl-authored, core-model#req:go-in-memory

**Scenario:** HCL parses into a directly usable in-memory model.
**Given** a model authored in HCL
**When** it is parsed by `polymodel-go`
**Then** it loads into the in-memory model that a Go consumer can use directly, with no serialized interchange file required.

### AC: validation-located
**Requirements:** core-model#req:validation

**Scenario:** Unresolved references fail with located errors.
**Given** a model containing an unresolved reference, such as a property reuse pointing at a property that does not exist
**When** the model is validated
**Then** validation fails with a located error that names the offending object and attribute.

## Out of Scope

These are explicitly **not** part of the core model and are deferred to separate
efforts:

- **Target generators** (inGitDB collection definitions, SQL, Firestore, language
  types) — downstream convergence work.
- **JSON/YAML serialization** of the model — HCL-first for v0; emit later only when a
  non-Go consumer needs it.
- **Catalog / remote-module resolution and `@pm:` references** — a separate SpecScore
  integration.
- **A PolyModel-native query language** — DTQL is the intended AST; v0 carries the
  query behind an opaque seam.
- **Migrations / schema evolution.**
- **Fuller type-system reconciliation** (localization, generic maps, additional
  constraints) — a separate type-system decision.

## Decisions

Four forks resolved during ideation are recorded as Decision artifacts (each with its
declined alternatives):

- Attribute model `property` / `field` / `column` (not one merged tabular type).
- No `Table` type — `Collection.Kind` (Editable | Computed).
- DTQL behind a placeholder query seam (no hard dependency on `dtql-go`).
- HCL-first authored format.

## Assumption Carryover

From the source Idea:

- **Survives:** the three-concept decoupling cleanly expresses real portfolio schemas
  (to be validated by modeling 2–3 real schemas in the v0 types).
- **Answered:** the DTQL dependency — resolved as a placeholder seam, so v0 no longer
  depends on `dtql-go` having a stable AST.
- **Survives:** `property`/`field`/`column` naming reduces confusion versus one shared
  term.
- **Narrowed:** Components embed into Entities only in v0.

## Open Questions

- The concrete shape of the placeholder query seam (opaque string vs minimal typed
  reference) is settled at implementation time, as long as a DTQL AST can replace it
  without reshaping the model.
- Whether `document` and `json` are aliases or distinct primitives is resolved during
  the type-system work.

---
*This document follows the https://specscore.md/feature-specification*
