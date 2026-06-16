---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: PolyModel Entity

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/entity?op=explore) | [Edit](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/entity?op=edit) | [Ask question](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/entity?op=ask) | [Request change](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/entity?op=request-change) |
**Status:** Approved
**Source Ideas:** —

## Summary

Entity: the semantic, identity-bearing business object with properties, components, and a key.

## Problem

The Entity is PolyModel's semantic anchor — the canonical definition of *what a thing
is*, independent of how it is stored or queried. Collections and Recordsets bind back
to Entity properties for meaning. Without a precise Entity concept, the model has no
authoritative home for identity, relationships, and canonical constraints. This
sub-feature specifies the Entity and its `property` attribute; shared rules (attribute
types, type system, binding precedence, validation) live in the parent
[core-model](../README.md).

## Behavior

### Properties and identity

#### REQ: entity-semantic

An **Entity** is a logical, identity-bearing business object — the semantic anchor.
An Entity MUST consist of zero or more **properties**, MAY embed **Component**s (per the
parent's `component-entities-only` rule), and MAY declare a **key** (one or more
property names) that identifies a record. An Entity carries no storage or query
concerns.

#### REQ: entity-property

A **property** is the canonical, semantic definition of an attribute. A property MUST
have a name and a type, MAY carry constraints (per the parent's `type-system` rule),
MAY be an **association** to another Entity (a relationship), and MAY **reuse** another
property's definition by reference. A property is the authoritative definition that
Collection fields and Recordset columns bind back to.

## Acceptance Criteria

### AC: entity-definition
**Requirements:** entity#req:entity-semantic, entity#req:entity-property

**Scenario:** An Entity with key, association, and property reuse validates.
**Given** an Entity declaring properties, an embedded Component, a key, a property that associates to another Entity, and a property that reuses another property's definition
**When** the model is validated
**Then** the Entity validates and the key, association, and reuse references resolve, and a key that names a property the Entity does not declare is rejected with a located error.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/feature-specification*
