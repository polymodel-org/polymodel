# PolyModel Catalog

## Overview

The PolyModel Catalog is an open ecosystem for discovering, sharing, versioning, and linking PolyModel modules, entities, datasets, and data sources.

The catalog is intended to complement the PolyModel specification and provide a practical mechanism for reuse and interoperability across organizations and projects.

## Core Vision

PolyModel defines canonical data models.

The Catalog allows those models to be:

- Published
- Discovered
- Versioned
- Referenced
- Linked together

A model should be definable once and reusable everywhere.

## Git-Native Architecture

PolyModel modules are published as Git repositories.

Example:

```text
github.com/polymodel-org/currency
github.com/polymodel-org/country
github.com/acme/payments
```

Versioning follows immutable Git tags:

```text
v0.1.0
v0.2.0
v1.0.0
```

The catalog is not the source of truth.

Repositories remain the source of truth.

The catalog acts as:

- Discovery service
- Documentation host
- Metadata index
- Dependency graph
- Caching proxy
- Checksum verification service

## Inspiration from Go Modules

PolyModel should borrow heavily from the Go module ecosystem.

Conceptually:

```text
Git Repository
    +
Semantic Version Tags
    =
Published PolyModel Module
```

Potential future services:

```text
catalog.polymodel.org
proxy.polymodel.org
sum.polymodel.org
```

## Canonical Entities

The catalog may host reusable standard entities.

Examples:

- Currency
- Country
- Address
- Person
- Company
- ExchangeRate

These entities become common reference points for interoperability.

## Dataset Registry

Datasets can publish mappings to canonical entities.

Example:

```text
Dataset: Orders

currency_code -> Currency.code
order_date    -> Date.value
```

```text
Dataset: ExchangeRates

currency_code -> Currency.code
date          -> Date.value
```

Because both datasets map to the same entities, tooling can automatically discover relationships.

## Automatic Dataset Linking

Example:

Orders:

```text
currency_code
order_date
amount
```

ExchangeRates:

```text
currency_code
date
rate
```

PolyModel tooling can infer:

```text
Orders.currency_code
        ↕
ExchangeRates.currency_code

Orders.order_date
        ↕
ExchangeRates.date
```

This enables automatic joins, enrichments, and navigation across datasets.

## Catalog Structure

Possible hierarchy:

```text
/entities
/datasets
/modules
/catalog
```

Examples:

```text
polymodel.org/entities/currency
polymodel.org/entities/country

polymodel.org/datasets/ecb-exchange-rates

polymodel.org/modules/github.com/acme/payments
```

## Why Keep It Under polymodel.org

Advantages:

- Stronger SEO
- Single authority domain
- Simpler user experience
- Stronger ecosystem branding
- Easier discovery of entities and datasets

## Long-Term Vision

PolyModel Catalog becomes an open ecosystem where:

- Models are composable
- Datasets are interoperable
- Common entities are standardized
- Data sources can be linked automatically

The network effect comes from shared semantics rather than shared storage.
