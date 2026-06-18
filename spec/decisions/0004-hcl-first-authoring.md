---
format: https://specscore.md/decision-specification
status: Approved
---

# Decision: HCL-first authored format

**Status:** Approved
**Date:** 2026-06-16
**Owner:** alex
**Tags:** —
**Source Idea:** core-model
**Supersedes:** —
**Superseded By:** —

## Context

`polymodel-go` v0 needs an authored on-disk format to parse. The PolyModel
specification expresses models in HCL, while the consuming tools in the portfolio
(DataTug, SpecScore, inGitDB) are YAML/JSON. We had to choose which format v0 parses.

## Decision

HCL-first: the authored format is HCL, matching the specification. v0 parses HCL into
the in-memory model; Go consumers use that in-memory model directly. JSON/YAML emission
is deferred (out of scope for v0).

## Rationale

HCL matches the canonical spec, so there is no divergence between what the spec shows
and what `polymodel-go` reads. The "consumers are YAML/JSON" concern does not actually
bind v0: the first Go consumers — the inGitDB schema generator and the SpecScore
`@pm:` resolver — use the in-memory model directly, not a serialized interchange file.
A serialization can be added later, when a non-Go consumer needs one.

## Declined Alternatives

### YAML/JSON-first authored format

Author models in YAML/JSON because the consuming tools are YAML/JSON. Lost: the spec is
HCL, and the Go consumers use the in-memory model rather than a serialized file, so
serialization is not a v0 constraint.

### JSON only for v0

Parse only JSON for v0, add HCL/YAML later. Lost: it diverges from the HCL spec from
day one; YAML/JSON can be added without contradicting the spec.

## Consequences at Decision Time

`polymodel-go` takes a dependency on `hashicorp/hcl/v2` for parsing. Non-Go consumers
must wait for JSON/YAML emission (out of scope for v0). The spec and the reference
parser stay in lockstep on format.

## Observed Consequences

None observed yet.

## Affected Features

- core-model

---
*This document follows the https://specscore.md/decision-specification*
