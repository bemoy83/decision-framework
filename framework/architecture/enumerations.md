# EV Decision Framework

## Enumerations

**Version:** 1.1
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the closed enumeration contract used throughout the EV Decision Framework.

Enumerations provide a controlled vocabulary for values that must remain stable across all framework implementations.

Supported implementations include:

* Reference Workbook
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* Future software implementations

Only values defined in this document shall be used where an enumeration is specified.

---

# Design Principles

Framework enumerations shall be:

* Stable
* Uppercase
* English
* Singular
* Machine-friendly
* Human-readable
* Technology independent

Enumeration values are part of the framework contract.

Display names may be localized.

Enumeration values shall never be translated.

---

# Enumeration Principles

The framework distinguishes between three concepts.

## Enumerations

Closed sets of values.

Examples:

* Availability
* Confidence
* Market

New values require framework versioning.

---

## Boolean Values

Simple true/false semantics.

Boolean fields shall use:

```text
TRUE
FALSE
```

No alternative values shall be used.

Examples of fields using Boolean values include:

* Active
* Weighted

---

## Domain Classifications

Some workbook columns contain classifications rather than enumerations.

Examples include:

* Review Category
* Equipment Category

These values belong to the domain model and are intentionally excluded from the enumeration contract.

Changes to domain classifications do not require changes to this document.

---

# Availability

Represents equipment availability.

| Value       | Description                        |
| ----------- | ---------------------------------- |
| STANDARD    | Included as standard equipment     |
| OPTIONAL    | Available as an optional extra     |
| PACKAGE     | Included through an option package |
| UNAVAILABLE | Not available                      |
| UNKNOWN     | Information unavailable            |

Availability is used by Equipment records.

---

# Confidence

Represents confidence in qualitative information.

| Value   | Description                             |
| ------- | --------------------------------------- |
| HIGH    | Multiple independent reliable sources   |
| MEDIUM  | Limited supporting evidence             |
| LOW     | Preliminary or weak supporting evidence |
| UNKNOWN | Insufficient information                |

Unknown is always preferred over unsupported assumptions.

Confidence applies to:

* Technical
* Equipment
* Evidence
* Reviews
* Hard Requirement Results

---

# Vehicle Status

Represents the lifecycle status of a vehicle.

| Value        |
| ------------ |
| ANNOUNCED    |
| PREORDER     |
| AVAILABLE    |
| DISCONTINUED |
| CANCELLED    |
| UNKNOWN      |

Vehicle Status applies only to Vehicle entities.

---

# Configuration Status

Represents the lifecycle status of a purchasable Configuration.

| Value        |
| ------------ |
| AVAILABLE    |
| UPCOMING     |
| DISCONTINUED |
| UNKNOWN      |

Configuration Status applies only to Configuration entities.

---

# Requirement Type

Defines how a Criterion participates in framework evaluation.

| Value         | Description                        |
| ------------- | ---------------------------------- |
| HARD          | Mandatory requirement              |
| WEIGHTED      | Contributes to scoring             |
| INFORMATIONAL | Recorded but not materially scored |
| EXCLUDED      | Explicitly excluded from scoring   |

Requirement Type belongs to Criterion definitions.

---

# Hard Requirement Result

Represents a Configuration's compliance result for one Hard Requirement Criterion.

| Value   | Description                                    |
| ------- | ----------------------------------------------- |
| PASS    | Configuration complies with the requirement     |
| FAIL    | Configuration does not comply with the requirement |
| UNKNOWN | Compliance has not been independently verified  |

Hard Requirement Result is used by HardRequirementResult records (ADR-009).

---

# Version Bump Type

Classifies why a Framework Version increment occurred.

| Value       | Description                                                    |
| ----------- | ---------------------------------------------------------------- |
| SCHEMA      | New/changed worksheet, column, or enumeration; no Criterion Weight/Type/Hard Requirement or calculation rule change |
| METHODOLOGY | Criterion Weight/Type/Hard Requirement flag or the calculation rule changed |
| NONE        | Entry did not change the active Framework Version               |

Version Bump Type is used by `11_DecisionLog` records (ADR-010).

---

# Source Type

Represents the origin of information.

| Value               |
| ------------------- |
| MANUFACTURER        |
| GOVERNMENT          |
| CERTIFICATION       |
| PROFESSIONAL_REVIEW |
| OWNER_REPORT        |
| COMMUNITY           |
| DATABASE            |
| VIDEO               |
| OTHER               |

Source Type classifies Sources only.

---

# Evidence Type

Represents the nature of documented evidence.

| Value            |
| ---------------- |
| SPECIFICATION    |
| MEASUREMENT      |
| OBSERVATION      |
| TEST_RESULT      |
| REVIEW_SUMMARY   |
| OWNER_EXPERIENCE |

Evidence Type classifies Evidence records.

---

# Market

Represents the intended sales market.

| Value  |
| ------ |
| NO     |
| SE     |
| DK     |
| FI     |
| EU     |
| UK     |
| GLOBAL |

Market applies to Configurations.

---

# Unit

Supported measurement units.

| Value     |
| --------- |
| MM        |
| CM        |
| M         |
| KM        |
| KMH       |
| KWH       |
| KWH_100KM |
| KW        |
| NM        |
| KG        |
| L         |
| DB        |

Additional units may be introduced through framework versioning.

---

# Framework Status

Represents the maturity of framework documentation.

| Value      |
| ---------- |
| DRAFT      |
| REVIEW     |
| LOCKED     |
| DEPRECATED |

Framework Status is used for framework documentation only.

---

# Decision Status

Represents the lifecycle of architectural and implementation decisions.

| Value       |
| ----------- |
| PROPOSED    |
| ACCEPTED    |
| IMPLEMENTED |
| SUPERSEDED  |

Decision Status applies to architectural governance.

---

# Reserved Enumerations

The following concepts are intentionally **not** framework enumerations.

## Review Category

Review categories are domain classifications.

They are defined by the framework methodology rather than the enumeration contract.

New Review categories do not require changes to this document.

---

## Equipment Category

Equipment categories are domain classifications.

They may evolve as new vehicle technologies emerge.

They are intentionally excluded from the enumeration contract.

---

## Criterion Category

Criterion categories are defined by the framework itself.

They are maintained through framework documentation rather than enumerations.

---

# Naming Rules

Enumeration values are immutable.

Once introduced, an enumeration value shall never be modified.

If the meaning of a value changes:

1. Introduce a new value.
2. Deprecate the previous value.
3. Preserve historical compatibility.

Enumeration values shall never be reused for different meanings.

---

# Validation Rules

Implementations shall validate that:

* only documented enumeration values are accepted;
* Boolean fields contain only TRUE or FALSE;
* enumeration values remain uppercase;
* undocumented values are rejected.

Validation shall occur before framework scoring.

---

# Implementation Notes

The Reference Workbook shall implement enumerations using workbook validation.

Future database implementations shall implement equivalent constraints using native technology.

All implementations shall preserve identical enumeration semantics.

---

# Guiding Principle

> **Enumerations define the controlled language of the framework. They are intentionally small, stable and technology independent, allowing every compliant implementation to validate data consistently while preserving long-term compatibility.**
