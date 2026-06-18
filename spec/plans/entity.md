---
format: https://specscore.md/plan-specification
status: Implemented
---
# Plan: Entity

**Status:** Implemented
**Source Feature:** core-model/entity
**Date:** 2026-06-16
**Owner:** alexandertrakhimenok
**Supersedes:** —

## Summary

Implement the Entity-specific behavior of `polymodel-go` v0: properties, identity key,
embedded Components, associations to other Entities, and property reuse. Builds on the
`core-model` foundations plan (model types, parser, validation).

## Approach

Test-driven Go, sequenced **after** the `core-model` foundations plan (the Entity types,
HCL parser, and validation framework come from there). This plan adds the Entity
semantics and their tests.

## Tasks

### Task 1: Entity semantics — properties, key, association, reuse

**Verifies:** core-model/entity#ac:entity-definition
**Depends-On:** —
**Status:** done

Implement an Entity with properties, an embedded Component, a `key` over property
names, a property association to another Entity, and property reuse by reference. Tests
assert these resolve and that a key naming a property the Entity does not declare is
rejected with a located error.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
