# EV Decision Framework

## Relationships Model

**Version:** 1.4
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the logical relationships between entities in the EV Decision Framework.

Relationships describe how entities reference one another while preserving ownership, traceability and data integrity.

The relationship model is implementation-independent and applies equally to spreadsheets, relational databases and future APIs.

---

# Guiding Principles

Relationships follow four principles.

1. Ownership is explicit.
2. References are explicit.
3. Data duplication should be minimized.
4. Information flows in one direction only.

---

# Ownership vs References

The framework distinguishes between ownership and references.

Ownership determines where information belongs.

References describe how entities use information owned elsewhere.

An entity should own only the information it creates.

An entity may reference information owned by another entity.

---

# Relationship Overview

```text
Vehicle
│
├── Configuration
│      │
│      ├── Technical
│      ├── Equipment
│      ├── Review
│      ├── Score
│      ├── OverallScore
│      └── HardRequirementResult
│
├── Technical
├── Review
└── Evidence

Equipment
    │
    ▼
EquipmentDefinition

Technical
    │
    ▼
TechnicalFieldDefinition

HardRequirementResult
    │
    ▼
Criterion (Type = HARD)

Evidence
    │
    ▼
Source

Review
    │
    ▼
Evidence

Score
    │
    ▼
Review

OverallScore
    │
    ▼
Score (aggregated)

FrameworkVersion
    │
    ▼
Score

Decision
    │
    ▼
FrameworkVersion
```

---

# Relationship Definitions

## Vehicle → Configuration

Relationship

**One-to-Many (1:N)**

A Vehicle may contain multiple purchasable Configurations.

Each Configuration belongs to exactly one Vehicle.

Examples

* Kia EV2

  * Exclusive
  * GT-Line

Vehicle represents the shared product.

Configuration represents the purchasable product.

Configuration owns its own lifecycle status.

Vehicle Status shall never be interpreted as Configuration Status.

---

## Vehicle → Technical

Relationship

**One-to-Many (1:N)**

Vehicle owns Technical records that are true for every Configuration.

Examples

* Wheelbase
* Platform
* Body Length

Configuration-specific specifications belong to Configuration instead.

---

## Configuration → Technical

Relationship

**One-to-Many (1:N)**

Configuration owns Technical records that differ between purchasable variants.

Examples

* Battery capacity
* Wheel size
* Charging capability

---

## Configuration → Equipment

Relationship

**One-to-Many (1:N)**

A Configuration may contain multiple Equipment records.

Equipment describes availability only.

Equipment never defines the feature itself.

---

## Equipment → EquipmentDefinition

Relationship

**Many-to-One (N:1)**

Every Equipment record references exactly one EquipmentDefinition.

EquipmentDefinition defines the feature.

Equipment defines whether that feature is available.

---

## Technical → TechnicalFieldDefinition

Relationship

**Many-to-One (N:1)**

Every Technical record references exactly one TechnicalFieldDefinition.

TechnicalFieldDefinition defines the field's name and canonical unit.

Technical defines the measured value, and an optional Qualifier when a measurement-condition variant (e.g. wheel size, mirror state) distinguishes it from another record of the same field for the same Vehicle or Configuration (ADR-007).

---

## Vehicle → Evidence

Relationship

**One-to-Many (1:N)**

Vehicle may own Evidence that applies to every Configuration.

Examples

* Measured cabin noise
* Crash test results
* Vehicle-wide efficiency measurements

---

## Configuration → Evidence

Relationship

**One-to-Many (1:N)**

Configuration may own Evidence describing trim-specific behaviour.

Examples

* Premium audio measurements
* Matrix LED observations
* 360 Camera behaviour

---

## Evidence → Source

Relationship

**Many-to-One (N:1)**

Every Evidence record references at least one Source.

Evidence always answers:

> What do we know?

Evidence never contains interpretation.

---

## Vehicle → Review

Relationship

**One-to-Many (1:N)**

Vehicle may contain Reviews describing characteristics shared across all Configurations.

Examples

* Ride quality
* Steering feel
* General cabin refinement

---

## Configuration → Review

Relationship

**One-to-Many (1:N)**

Configuration may contain Reviews describing trim-specific characteristics.

Examples

* Premium audio quality
* Camera usability
* Interior equipment

---

## Review → Evidence

Relationship

**One-to-Many (1:N)**

Every Review references one or more Evidence records.

Evidence supports the Review.

Reviews interpret Evidence.

Evidence does not reference Reviews.

This relationship intentionally preserves a one-way flow of information.

Future framework versions may replace this with a more normalized many-to-many model if required.

---

## Score → Review

Relationship

**Many-to-One (N:1)**

Every Criterion Score references one Review.

Scores evaluate Reviews.

Reviews do not evaluate Scores.

---

## Score → FrameworkVersion

Relationship

**Many-to-One (N:1)**

Every Score references the Framework Version used during evaluation.

This guarantees reproducibility.

---

## Configuration → OverallScore

Relationship

**One-to-Many (1:N)**

A Configuration may have one OverallScore per Framework Version.

OverallScore aggregates that Configuration's Criterion Scores; it does not introduce new information.

---

## Score → OverallScore

Relationship

**Aggregation, not reference**

OverallScore is computed from the set of Criterion Scores belonging to a Configuration under a Framework Version.

OverallScore does not reference a single Score or a single Review.

Every OverallScore carries a coverage percentage describing what proportion of the framework's weighted criteria contributed to it.

---

## Configuration → HardRequirementResult

Relationship

**One-to-Many (1:N)**

A Configuration has one HardRequirementResult per Hard Requirement Criterion.

HardRequirementResult never contributes to weighted scoring.

---

## HardRequirementResult → Criterion

Relationship

**Many-to-One (N:1)**

Every HardRequirementResult references exactly one Criterion, restricted to Criteria where `Type = HARD`.

Criterion defines the requirement. HardRequirementResult defines whether a specific Configuration complies with it.

---

## Decision → FrameworkVersion

Relationship

**Many-to-One (N:1)**

Every architecture decision references the Framework Version under which it became effective.

---

# Ownership Summary

| Entity              | Owns                                                                                           |
| ------------------- | ---------------------------------------------------------------------------------------------- |
| Vehicle             | Shared identity, shared Technical, shared Evidence, shared Reviews                             |
| Configuration       | Purchasable characteristics, status, Equipment, configuration-specific Technical, Evidence and Reviews |
| TechnicalFieldDefinition | Technical field name and canonical unit                                                  |
| EquipmentDefinition | Feature definition                                                                             |
| Equipment           | Availability                                                                                   |
| Source              | Information origin                                                                             |
| Evidence            | Verified observations                                                                          |
| Review              | Interpretation                                                                                 |
| Score               | Evaluation                                                                                     |
| HardRequirementResult | Hard Requirement compliance conclusion                                                       |
| OverallScore        | Aggregated evaluation, coverage percentage                                                      |
| FrameworkVersion    | Framework identity                                                                             |

---

# Referential Integrity

Every implementation shall enforce the following relationships.

A Configuration shall reference one Vehicle.

Technical shall reference either one Vehicle or one Configuration, and one TechnicalFieldDefinition.

Equipment shall reference:

* one Configuration;
* one EquipmentDefinition.

Evidence shall reference:

* one Vehicle or one Configuration;
* at least one Source.

Review shall reference:

* one Vehicle or one Configuration;
* at least one Evidence record.

Score shall reference:

* one Configuration;
* one Review;
* one FrameworkVersion.

OverallScore shall reference:

* one Configuration;
* one FrameworkVersion.

OverallScore shall not reference a single Review or a single Score.

HardRequirementResult shall reference:

* one Configuration;
* one Criterion, where `Type = HARD`.

Broken references are considered implementation errors.

---

# Information Flow

Information always flows forward.

```text
Source
    │
    ▼
Evidence
    │
    ▼
Review
    │
    ▼
Criterion Score
    │
    ▼
Overall Score
```

No relationship should introduce reverse information flow.

Scores never modify Reviews.

Reviews never modify Evidence.

Evidence never modifies Sources.

---

# Unknown Values

Unknown information remains Unknown until supported by Evidence.

Unknown values shall never be replaced by assumptions.

---

# Future Compatibility

The relationship model supports future migration to:

* Excel
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* GraphQL

without conceptual redesign.

Future framework versions may introduce additional normalization where justified by real implementation experience.

---

# Guiding Principle

> **Entities own information. Relationships create knowledge. References preserve traceability.**
