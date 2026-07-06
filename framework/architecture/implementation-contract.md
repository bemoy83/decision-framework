# EV Decision Framework

## Implementation Contract

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines the minimum requirements that any implementation of the EV Decision Framework must satisfy.

The purpose is to ensure that different implementations remain functionally equivalent while allowing freedom in technology and implementation details.

Examples of valid implementations include:

* Excel workbook
* Google Sheets
* SQLite
* PostgreSQL
* REST API
* Web application

Implementations may differ internally.

They must produce equivalent framework behaviour.

---

# Source of Truth

The implementation shall follow the framework documentation.

The implementation shall **never** redefine framework behaviour.

Documentation always takes precedence over implementation.

---

# Required Components

Every implementation shall support:

* Vehicles
* Configurations
* Technical Data
* Equipment
* Equipment Definitions
* Evidence
* Reviews
* Sources
* Criteria
* Scores
* Framework Versions

These concepts are mandatory.

---

# Entity Integrity

Every entity shall have:

* a stable unique identifier;
* a clearly defined owner;
* explicit relationships;
* a documented lifecycle.

No implementation shall duplicate entity ownership.

---

# Referential Integrity

Every implementation shall guarantee valid references.

Minimum requirements include:

* Every Configuration references an existing Vehicle.
* Every Technical record references an existing Vehicle or Configuration.
* Every Equipment record references an existing Equipment Definition.
* Every Evidence record references at least one Source.
* Every Review references at least one Evidence record.
* Every Criterion Score references:

  * one Criterion,
  * one Review,
  * one Framework Version.
* Every Overall Score references:

  * one Configuration,
  * one Framework Version.

Broken references are considered implementation errors.

---

# Data Ownership

Each implementation shall preserve ownership boundaries.

Examples:

Vehicle owns:

* identity
* manufacturer
* model

Technical Data owns:

* measurable specifications

Evidence owns:

* documented observations

Review owns:

* interpretation

Score owns:

* evaluation

Implementations shall not duplicate ownership across entities.

---

# Enumerations

Implementations shall use the enumeration values defined in:

`framework/architecture/enumerations.md`

Enumeration values are considered part of the public framework contract.

Implementations may localize display text.

Internal values shall remain unchanged.

---

# Framework Versioning

Every evaluation shall reference the Framework Version used.

Implementations shall support historical evaluations without recalculation.

Historical results must remain reproducible.

---

# Traceability

Every recommendation shall be fully traceable.

Minimum traceability chain:

```text
Recommendation
    ↓
Overall Score
    ↓
Criterion Score
    ↓
Review
    ↓
Evidence
    ↓
Source
```

If traceability is broken, the implementation is not framework compliant.

---

# Unknown Values

Unknown information shall remain Unknown.

Implementations shall never replace missing information with inferred values unless explicitly documented.

---

# Confidence

Every qualitative assessment shall support confidence values.

Confidence shall propagate through the evaluation process.

Implementations shall not silently discard confidence information.

---

# Scoring

The implementation shall calculate:

* Criterion Scores
* Overall Scores

The implementation shall not permit manual editing of calculated scores.

Raw data may be edited.

Calculated results shall be derived from framework rules.

---

# Explainability

Every calculated score shall be explainable.

Implementations shall expose sufficient information to identify:

* contributing reviews;
* supporting evidence;
* originating sources.

No score shall exist without an explanation path.

---

# Determinism

Given:

* identical framework version,
* identical data,
* identical evidence,

every compliant implementation shall produce identical scores.

Framework behaviour shall be deterministic.

---

# Validation

Implementations should validate:

* required identifiers;
* enumeration values;
* mandatory references;
* duplicate identifiers;
* orphaned records;
* missing framework versions.

Validation errors should be reported before scoring.

---

# Migration

Implementations should permit export without loss of meaning.

The framework shall remain portable between:

* spreadsheets;
* databases;
* APIs.

No implementation-specific behaviour should become part of the framework.

---

# Non-Goals

The implementation contract does not specify:

* user interface;
* programming language;
* storage engine;
* performance optimisation;
* deployment model.

These are implementation choices.

---

# Compliance Levels

## Fully Compliant

Implements every mandatory requirement.

Produces deterministic, explainable and reproducible results.

---

## Partially Compliant

Implements the framework but lacks optional validation or migration features.

---

## Non-Compliant

Violates one or more mandatory framework requirements.

Examples include:

* broken traceability;
* duplicated ownership;
* undocumented scoring;
* missing framework version binding.

---

# Guiding Principle

Implementations are replaceable.

The framework is not.

Technology may evolve.

The implementation contract ensures that every implementation remains faithful to the methodology rather than introducing its own behaviour.
