# ADR-004 — Configuration Lifecycle Status

**Status:** Accepted

**Date:** 2026-07-06

**Framework Version:** 1.2 (Next)

**Supersedes:** None

**Related:**

* ADR-003 — Configuration Is The Evaluated Entity
* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md
* framework/architecture/enumerations.md

---

# Context

ADR-003 established the Configuration entity as the primary evaluation target of the framework.

A Configuration represents the smallest purchasable product that can receive an independent recommendation.

During implementation of the first reference dataset, it became apparent that the framework can represent the lifecycle state of a Vehicle, but not the lifecycle state of an individual Configuration.

Examples include:

* a Vehicle remaining available while one Configuration is discontinued;
* a new Configuration becoming available before others;
* temporary market-specific availability.

The framework already defines a **Configuration Status** enumeration.

However, the current workbook schema does not expose a corresponding `Status` attribute on the `Configuration` entity.

This creates an inconsistency between the architectural model and its implementation.

---

# Decision

The Configuration entity shall own a lifecycle `Status` attribute.

The attribute represents the commercial availability of the purchasable Configuration.

It is distinct from Vehicle Status.

Vehicle Status describes the lifecycle of the vehicle model.

Configuration Status describes the lifecycle of a specific purchasable Configuration.

These two concepts shall remain independent.

---

# Rationale

Configuration is the framework's primary evaluation target.

A recommendation can only be produced for a Configuration that exists as a purchasable product.

Commercial lifecycle is therefore part of Configuration identity rather than Technical information.

Status is not:

* a measurable property;
* an opinion;
* Evidence;
* Review;
* Technical data.

Status describes whether the Configuration is currently available for evaluation and purchase.

It therefore belongs to the Configuration entity.

---

# Consequences

The following architectural changes shall be introduced.

## Entity Model

Configuration owns:

* Status

Vehicle continues to own:

* Vehicle Status

These attributes represent different concepts and shall never be derived from one another.

---

## Workbook Schema

Worksheet:

```text
03_Configurations
```

shall include:

```text
Status
```

using the existing framework enumeration:

```text
Configuration Status
```

---

## Enumeration Contract

No changes are required.

`Configuration Status` already exists within the framework enumeration contract.

ADR-004 consumes an existing enumeration rather than introducing a new one.

---

## Implementation Contract

Reference Workbook implementations shall validate the new Status column using the existing `Configuration Status` enumeration.

No other worksheet shall own Configuration lifecycle state.

---

# Alternatives Considered

## Option A

Derive Configuration Status from Vehicle Status.

Rejected.

Vehicle lifecycle does not imply Configuration lifecycle.

A Vehicle may remain available while individual Configurations are discontinued or introduced.

---

## Option B

Store Status as Technical data.

Rejected.

Lifecycle state is not a measurable technical fact.

Technical entities own objective specifications.

Commercial availability belongs to Configuration identity.

---

## Option C

Do not represent Configuration lifecycle.

Rejected.

The framework evaluates purchasable products.

The lifecycle of those products is therefore architecturally significant.

---

# Migration Strategy

Framework Version 1.1 remains valid without Configuration Status.

Framework Version 1.2 introduces the new attribute.

Existing datasets may initially contain:

```text
UNKNOWN
```

until verified.

No historical data shall be invalidated.

---

# Impact

Affected documents:

* entity-model.md
* workbook-schema.md
* implementation-contract.md

Affected workbook:

* `03_Configurations`

Affected implementations:

* Reference Workbook
* Future database implementations
* Future API implementations

No changes are required to:

* Technical
* Equipment
* Evidence
* Reviews
* Scoring

---

# Guiding Principle

> **A Configuration represents the smallest purchasable product that can receive an independent recommendation. Its lifecycle state is therefore part of its identity and shall be represented explicitly rather than inferred from the lifecycle of its parent Vehicle.**
