# EV Decision Framework

## Implementation Contract

**Version:** 1.2
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the minimum requirements that every implementation of the EV Decision Framework must satisfy.

Its purpose is to ensure that different implementations produce equivalent framework behaviour while remaining free to choose different technologies.

Valid implementations include:

* Excel
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* Web applications

Implementations may differ internally.

They shall produce equivalent framework behaviour.

---

# Scope

This contract specifies:

* mandatory entities;
* mandatory relationships;
* mandatory traceability;
* mandatory framework behaviour.

It deliberately does **not** specify implementation technology.

---

# Source of Truth

Framework documentation is the authoritative source of truth.

Implementations shall implement the framework.

Implementations shall never redefine framework behaviour.

If implementation and documentation disagree, documentation takes precedence until explicitly updated through the framework governance process.

---

# Required Entities

Every compliant implementation shall support the following entities.

```text id="bhmjlwm"
Vehicle
Configuration
Technical
Criterion
EquipmentDefinition
Equipment
Source
Evidence
Review
Score
FrameworkVersion
Decision
```

These entities are mandatory.

Implementations may extend the model but shall not redefine mandatory entities.

---

# Entity Ownership

Every entity shall own only the information it creates.

Ownership shall never be duplicated.

Summary

| Entity              | Owns                    |
| ------------------- | ----------------------- |
| Vehicle             | Shared identity         |
| Configuration       | Purchasable identity    |
| Technical           | Measurable facts        |
| EquipmentDefinition | Feature definition      |
| Equipment           | Feature availability    |
| Source              | Information origin      |
| Evidence            | Documented observations |
| Review              | Interpretation          |
| Criterion           | Evaluation definition   |
| Score               | Calculated evaluation   |
| FrameworkVersion    | Framework identity      |
| Decision            | Architectural history   |

Implementations shall preserve these ownership boundaries.

---

# Identifier Integrity

Every entity shall provide a stable unique identifier.

Minimum identifiers include:

* VehicleID
* ConfigurationID
* TechnicalID
* CriterionID
* EquipmentDefinitionID
* EquipmentID
* SourceID
* EvidenceID
* ReviewID
* ScoreID
* FrameworkVersion
* DecisionID

Identifiers shall remain stable throughout the lifetime of the entity.

---

# Referential Integrity

Every implementation shall guarantee valid references.

Mandatory relationships include:

Configuration references one Vehicle.

Technical references either:

* one Vehicle; or
* one Configuration.

Equipment references:

* one Configuration;
* one EquipmentDefinition.

Evidence references:

* one Vehicle or one Configuration;
* at least one Source.

Review references:

* one Vehicle or one Configuration;
* at least one Evidence record.

Score references:

* one Configuration;
* one Criterion;
* one Review;
* one FrameworkVersion.

Decision references one FrameworkVersion.

Broken references are implementation errors.

---

# Configuration as Evaluation Target

Configuration is the framework's primary evaluation target.

Scores shall always belong to Configurations.

Vehicle may contribute shared information.

Vehicle shall never receive an Overall Score.

Purchase recommendations shall always refer to Configurations.

Every Configuration shall own exactly one Configuration Status.

Configuration Status shall be validated using the Configuration Status enumeration.

Vehicle Status shall never be used as a substitute for Configuration Status.

---

# Enumerations

Implementations shall use the enumeration values defined in:

```text id="1ejh9z"
framework/architecture/enumerations.md
```

Enumeration identifiers are part of the public framework contract.

Display values may be localized.

Internal values shall remain unchanged.

---

# Framework Versioning

Every evaluation shall reference one FrameworkVersion.

FrameworkVersion guarantees:

* reproducibility;
* historical comparison;
* methodology traceability.

Historical evaluations shall never be recalculated automatically using newer framework versions.

---

# Information Flow

Implementations shall preserve the following information flow.

```text id="hg3rgm"
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

Reverse information flow is prohibited.

Scores shall never modify Reviews.

Reviews shall never modify Evidence.

Evidence shall never modify Sources.

---

# Traceability

Every recommendation shall be fully explainable.

Minimum traceability chain:

```text id="z5yptl"
Recommendation
        │
        ▼
Overall Score
        │
        ▼
Criterion Score
        │
        ▼
Review
        │
        ▼
Evidence
        │
        ▼
Source
```

Every score shall expose a complete explanation path.

Loss of traceability results in framework non-compliance.

---

# Unknown Values

Unknown information shall remain Unknown.

Implementations shall never replace missing information with inferred values unless explicitly documented by the framework.

Unknown values shall not participate in scoring unless framework rules define explicit behaviour.

---

# Confidence

Evidence and Reviews shall support confidence values.

Confidence shall remain visible throughout the evaluation pipeline.

Implementations shall not silently discard confidence information.

---

# Scoring

Implementations shall calculate:

* Criterion Scores;
* Overall Scores.

Calculated scores shall never be entered manually.

Scores shall always be derived from:

* Reviews;
* Criteria;
* Framework weighting;
* FrameworkVersion.

---

# Explainability

Every calculated score shall explain:

* which Criterion was evaluated;
* which Review contributed;
* which Evidence supported the Review;
* which Sources supported the Evidence.

No score shall exist without a complete explanation path.

---

# Determinism

Given:

* identical FrameworkVersion;
* identical data;
* identical Evidence;
* identical Reviews;

every compliant implementation shall produce identical results.

Framework behaviour shall be deterministic.

---

# Validation

Implementations should validate:

* unique identifiers;
* mandatory relationships;
* enumeration values;
* duplicate identifiers;
* orphaned records;
* missing FrameworkVersion references.

Validation should occur before scoring.

---

# Migration

Implementations should permit migration without loss of meaning.

Supported migration targets include:

* spreadsheets;
* databases;
* APIs.

Implementation-specific behaviour shall never become part of the framework definition.

---

# Non-Goals

This contract does not define:

* user interface;
* storage technology;
* programming language;
* performance optimisation;
* deployment strategy.

These remain implementation choices.

---

# Compliance Levels

## Fully Compliant

Implements every mandatory framework requirement.

Produces deterministic, explainable and reproducible results.

---

## Partially Compliant

Implements the framework correctly but omits optional validation or migration capabilities.

---

## Non-Compliant

Violates one or more mandatory framework requirements.

Examples include:

* broken traceability;
* duplicated ownership;
* undocumented scoring;
* missing FrameworkVersion binding;
* invalid entity relationships.

---

# Guiding Principle

> **Implementations may differ internally, but they shall always preserve the framework's methodology, ownership, traceability and behaviour.**
