# EV Decision Framework

## Workbook Schema

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines the reference workbook schema for the EV Decision Framework.

The workbook is the reference implementation of the framework.

It is **not** the source of truth.

The source of truth is the framework documentation.

The workbook exists to implement the framework consistently.

---

# Design Principles

The workbook should:

* Separate facts from evaluations.
* Avoid duplicate data.
* Use unique identifiers for relationships.
* Keep calculations separate from source data.
* Be easily migrated to a relational database.
* Remain understandable without macros or external tools.

---

# Workbook Structure

The workbook consists of the following worksheets.

| Worksheet               | Purpose                           |
| ----------------------- | --------------------------------- |
| README                  | Workbook metadata and version     |
| 01_Criteria             | Evaluation criteria and weighting |
| 02_Vehicles                 | Master list of vehicles           |
| 03_Configurations       | Market-specific trims and pricing |
| 04_Technical            | Technical specifications          |
| 05_Equipment            | Equipment availability            |
| 06_EquipmentDefinitions | Equipment catalogue               |
| 07_Reviews              | Qualitative assessments           |
| 08_Evidence             | Supporting evidence               |
| 09_Sources              | Source catalogue                  |
| 10_Scoring              | Calculated scores                 |
| 11_DecisionLog          | Framework and workbook history    |

---

# Worksheet Specifications

## README

Purpose

Workbook metadata.

Contains:

* Workbook version
* Framework version
* Last updated
* Domain
* Notes

No calculations.

---

## 01_Criteria

Purpose

Defines every criterion used by the framework.

Primary Key

CriterionID

Columns

* CriterionID
* Category
* Name
* Description
* Type
* Weight
* HardRequirement
* Active
* Notes

Only framework versions modify this worksheet.

---

## 02_Vehicles

Purpose

Master vehicle registry.

Primary Key

VehicleID

Columns

* VehicleID
* Manufacturer
* Model
* ModelYear
* Platform
* Status
* Notes

Contains one row per vehicle.

No technical specifications belong here.

---

## 03_Configurations

Purpose

Defines market-specific variants.

Primary Key

ConfigurationID

Relationship

Car → Configuration

1:N

Columns

* ConfigurationID
* VehicleID
* Market
* Trim
* BasePrice
* ConfiguredPrice
* Notes

---

## 04_Technical

Purpose

Stores immutable technical specifications.

Relationship

Car → Technical Data

1:N

Each row represents one measurable specification.

Columns

* VehicleID
* TechnicalField
* Value
* Unit
* SourceID
* Confidence
* LastUpdated

Examples

* Length
* Width
* Battery Gross
* Battery Net
* WLTP
* Charging Speed

---

## 05_Equipment

Purpose

Stores equipment availability.

Relationship

Car → Equipment

1:N

Columns

* VehicleID
* EquipmentID
* Availability
* Configuration
* SourceID
* Confidence

Availability values are defined in Enumerations.

---

## 06_EquipmentDefinitions

Purpose

Master catalogue of equipment.

Primary Key

EquipmentID

Columns

* EquipmentID
* Category
* Name
* Description
* Weighted
* Notes

Examples

* 360 Camera
* Matrix LED
* OTA Updates
* Ambient Lighting

---

## 07_Reviews

Purpose

Stores qualitative evaluations.

Relationship

Car → Review

1:N

Columns

* ReviewID
* VehicleID
* Category
* Score
* Confidence
* Summary
* SourceID

Reviews should summarise evidence.

They are not evidence themselves.

---

## 08_Evidence

Purpose

Stores supporting observations.

Relationship

Car → Evidence

1:N

Columns

* EvidenceID
* VehicleID
* Claim
* Evidence
* SourceID
* Confidence

Evidence should be factual whenever possible.

Examples

Claim

Low cabin noise

Evidence

67 dB measured at 110 km/h

---

## 09_Sources

Purpose

Central source registry.

Primary Key

SourceID

Columns

* SourceID
* Type
* Title
* Publisher
* URL
* PublishedDate
* RetrievedDate
* Notes

Sources should never be duplicated.

---

## 10_Scoring

Purpose

Calculated output.

Contains no manually entered scores.

Columns

* VehicleID
* CriterionID
* RawScore
* WeightedScore
* Explanation

Scores should be generated from framework rules.

---

## 11_DecisionLog

Purpose

Maintains project history.

Columns

* Date
* Version
* Decision
* Reason
* Reference

Every significant framework change should be documented.

---

# Data Relationships

The workbook follows a relational structure.

```text
Car
├── Configurations
├── Technical
├── Equipment
├── Reviews
├── Evidence
└── Scores

Source
├── Technical
├── Equipment
├── Reviews
└── Evidence

Criteria
└── Scores
```

---

# Data Ownership

Each worksheet owns its data.

Example

Vehicle length exists only in **04_Technical**.

It must never be duplicated elsewhere.

---

# Calculations

Calculated values belong only in:

* 10_Scoring
* Dashboard (future)

Source worksheets should never contain derived values.

---

# Data Entry Rules

When entering data:

1. Always reference an existing VehicleID.
2. Always reference an existing SourceID where applicable.
3. Never overwrite historical evidence.
4. Unknown is preferable to assumed.
5. Confidence must be specified for qualitative information.

---

# Naming Conventions

Identifiers should be stable.

Examples

VehicleID

* KIA_EV2
* RENAULT_4
* SKODA_ELROQ

EquipmentID

* EQ_360_CAMERA
* EQ_MATRIX_LED
* EQ_OTA

CriterionID

* CR_DRIVING
* CR_TECHNOLOGY
* CR_VALUE

---

# Future Compatibility

The workbook is intentionally designed to support future migration to:

* SQLite
* PostgreSQL
* Web applications
* APIs
* Automated data collection

No worksheet should rely on spreadsheet-specific behaviour that cannot be represented in a relational database.

---

# Reference Implementation

The workbook is the reference implementation of the EV Decision Framework.

Future implementations may use different technologies.

They should implement this schema without changing the framework methodology.
