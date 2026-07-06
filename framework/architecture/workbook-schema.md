# EV Decision Framework

## Workbook Schema

**Version:** 1.4
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the structure of the Reference Workbook.

The workbook is the official reference implementation of the EV Decision Framework.

Its purpose is to demonstrate how the framework architecture is implemented in a spreadsheet while remaining independent of spreadsheet-specific features.

The workbook implements the framework.

It does not define the framework.

The authoritative source of truth remains the framework documentation.

---

# Scope

This document specifies:

* workbook structure;
* worksheet responsibilities;
* worksheet relationships;
* worksheet ownership;
* identifier usage;
* implementation constraints.

It does not define:

* evaluation methodology;
* scoring philosophy;
* framework architecture.

Those concepts are defined elsewhere within the framework documentation.

---

# Design Principles

The Reference Workbook follows the same architectural principles as the framework itself.

## 1. Documentation First

The workbook shall implement the documented framework.

Documentation always takes precedence over implementation.

---

## 2. One Worksheet — One Responsibility

Each worksheet owns one clearly defined concept.

Examples include:

* Vehicle
* Configuration
* Technical
* Equipment
* Evidence
* Review

No worksheet should own multiple unrelated concepts.

---

## 3. Explicit Relationships

Relationships are represented using stable identifiers.

Examples include:

* VehicleID
* ConfigurationID
* TechnicalID
* EquipmentDefinitionID
* EquipmentID
* EvidenceID
* ReviewID

Worksheet order never implies relationships.

Relationships are always explicit.

---

## 4. Minimal Duplication

Information shall be stored only once whenever practical.

Shared information belongs to Vehicle.

Purchasable differences belong to Configuration.

Feature definitions belong to EquipmentDefinition.

Availability belongs to Equipment.

---

## 5. Separation of Knowledge

The workbook intentionally separates different kinds of information.

Identity

↓

Facts

↓

Availability

↓

Observations

↓

Interpretation

↓

Evaluation

↓

Recommendation

No worksheet should combine multiple stages.

---

## 6. Explainability

Every calculated result shall be traceable back to:

* Review;
* Evidence;
* Source.

No score shall exist without an explanation path.

---

## 7. Reproducibility

Every evaluation shall reference the FrameworkVersion used.

Historical evaluations shall remain reproducible.

---

## 8. Technology Independence

The workbook is a reference implementation.

Its structure has been intentionally designed to support future migration to:

* Google Sheets;
* SQLite;
* PostgreSQL;
* REST APIs;
* Web applications.

No worksheet should depend on spreadsheet-specific behaviour that cannot reasonably be represented elsewhere.

---

# Workbook Structure

The workbook consists of the following worksheets.

| Worksheet               | Type        | Purpose                                          |
| ----------------------- | ----------- | ------------------------------------------------ |
| README                  | Metadata    | Workbook metadata and implementation information |
| 01_Criteria             | Reference   | Framework criteria and weighting definitions     |
| 02_Vehicles             | Operational | Shared vehicle identity                          |
| 03_Configurations       | Operational | Purchasable vehicle configurations               |
| 04_Technical            | Operational | Measurable technical characteristics             |
| 05_Equipment            | Operational | Equipment availability by configuration          |
| 06_EquipmentDefinitions | Reference   | Equipment catalogue                              |
| 07_Reviews              | Operational | Qualitative interpretation of evidence           |
| 08_Evidence             | Operational | Documented observations                          |
| 09_Sources              | Reference   | Information source catalogue                     |
| 10_Scoring              | Calculated  | Criterion scores                                 |
| 11_DecisionLog          | Metadata    | Framework and workbook change history            |
| 12_OverallScores        | Calculated  | Aggregated Overall Score and coverage per Configuration |

---

# Worksheet Categories

The workbook distinguishes between four worksheet categories.

## Metadata Worksheets

Describe the workbook itself.

Examples:

* README
* DecisionLog

Metadata worksheets are not part of the evaluation model.

---

## Reference Worksheets

Contain reusable framework definitions.

Examples:

* Criteria
* Equipment Definitions
* Sources

Reference worksheets define concepts shared by many operational records.

Reference worksheets should change infrequently.

---

## Operational Worksheets

Contain domain-specific evaluation data.

Examples:

* Vehicles
* Configurations
* Technical
* Equipment
* Evidence
* Reviews

Operational worksheets represent the working dataset of the framework.

---

## Calculated Worksheets

Contain generated output.

Examples:

* Scoring
* Overall Scores

Calculated worksheets shall never become the authoritative source for raw information.

Every calculated value must remain reproducible from the underlying operational data.

---

# Workbook Architecture

The workbook mirrors the logical architecture of the framework.

```text
Vehicle
        │
        ▼
Configuration
        │
        ├────────────┐
        ▼            ▼
Technical      Equipment
        │            │
        └──────┬─────┘
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

Worksheet ordering follows the natural progression of information through the framework.

Relationships are always established using identifiers rather than worksheet position.

**Note**

The workbook diagram illustrates the primary evaluation pipeline.

Not every Evidence record originates from Technical or Equipment.

Evidence may also represent documented observations, independent measurements or external findings supported by one or more Sources.

The complete information flow is defined in `framework/architecture/data-flow.md`.

---

# Workbook Philosophy

The Reference Workbook is intended to remain:

* understandable;
* inspectable;
* explainable;
* deterministic;
* portable.

The workbook favours architectural clarity over spreadsheet optimisation.

Whenever trade-offs exist, preserving framework methodology takes precedence over implementation convenience.

---

# Worksheet Specifications

---

# README

## Purpose

Stores workbook metadata.

This worksheet documents the Reference Workbook itself rather than evaluation data.

README provides versioning and implementation information and shall not participate in scoring.

---

## Worksheet Type

Metadata

---

## Primary Key

None

---

## References

None

---

## Referenced By

None

---

## Columns

| Column           | Description                                   |
| ---------------- | --------------------------------------------- |
| WorkbookVersion  | Version of the reference workbook             |
| FrameworkVersion | Framework version implemented by the workbook |
| LastUpdated      | Date of last workbook revision                |
| Domain           | Framework domain (e.g. Electric Vehicles)     |
| Status           | Workbook lifecycle status                     |
| Notes            | Optional implementation notes                 |

---

## Notes

README contains no calculations.

README is maintained manually.

---

# 01_Criteria

## Purpose

Defines every evaluation criterion used by the framework.

Criteria describe **what** is evaluated.

They never contain evaluation results.

---

## Worksheet Type

Reference

---

## Primary Key

CriterionID

---

## References

None

---

## Referenced By

* 10_Scoring

---

## Columns

| Column          | Description                               |
| --------------- | ----------------------------------------- |
| CriterionID     | Stable criterion identifier               |
| Category        | Criterion category                        |
| Name            | Criterion name                            |
| Description     | Criterion description                     |
| Type            | Requirement type                          |
| Weight          | Criterion weighting                       |
| HardRequirement | Mandatory requirement indicator           |
| Active          | Indicates whether the criterion is active |
| Notes           | Optional framework notes                  |

---

## Notes

Criteria are framework definitions.

Changes should occur only when the framework itself evolves.

Historical scoring should remain reproducible across framework versions.

---

# 02_Vehicles

## Purpose

Represents the shared identity of a vehicle model.

Vehicles group one or more purchasable Configurations.

Vehicles are **not** directly ranked.

---

## Worksheet Type

Operational

---

## Primary Key

VehicleID

---

## References

None

---

## Referenced By

* 03_Configurations
* 04_Technical
* 07_Reviews
* 08_Evidence

---

## Columns

| Column       | Description               |
| ------------ | ------------------------- |
| VehicleID    | Stable vehicle identifier |
| Manufacturer | Vehicle manufacturer      |
| Model        | Model name                |
| ModelYear    | Model year                |
| Platform     | Vehicle platform          |
| Status       | Lifecycle status          |
| Notes        | Optional notes            |

---

## Notes

Vehicles contain only information shared across all Configurations.

Configuration-specific information shall not be stored here.

Technical specifications belong in **04_Technical**.

Equipment belongs in **05_Equipment**.

Pricing belongs in **03_Configurations**.

---

# 03_Configurations

## Purpose

Represents purchasable vehicle configurations.

Configuration is the framework's primary evaluation target.

Every purchasing recommendation refers to a Configuration.

---

## Worksheet Type

Operational

---

## Primary Key

ConfigurationID

---

## References

* VehicleID

---

## Referenced By

* 04_Technical
* 05_Equipment
* 07_Reviews
* 08_Evidence
* 10_Scoring

---

## Columns

| Column          | Description                      |
| --------------- | -------------------------------- |
| ConfigurationID | Stable configuration identifier  |
| VehicleID       | Parent vehicle                   |
| Market          | Market or region                 |
| Status          | Commercial lifecycle status      |
| Trim            | Trim designation                 |
| BasePrice       | Manufacturer base price          |
| ConfiguredPrice | Price of evaluated configuration |
| Notes           | Optional notes                   |

---

## Notes

Each Configuration belongs to exactly one Vehicle.

Configuration-specific pricing is stored only here.

Configuration acts as the purchasing entity defined in ADR-003.

Status values shall follow the **Configuration Status** framework enumeration.

Configuration Status describes the commercial lifecycle of the purchasable Configuration and shall never be derived from Vehicle Status (see ADR-004).

---

# 04_Technical

## Purpose

Stores measurable technical characteristics.

Technical data represents verified factual information.

Technical never contains interpretation or evaluation.

---

## Worksheet Type

Operational

---

## Primary Key

TechnicalID

---

## References

* VehicleID or ConfigurationID
* SourceID

---

## Referenced By

* 08_Evidence

---

## Columns

| Column          | Description                     |
| --------------- | ------------------------------- |
| TechnicalID     | Stable technical identifier     |
| VehicleID       | Parent vehicle (optional)       |
| ConfigurationID | Parent configuration (optional) |
| Property        | Technical property name         |
| Value           | Measured value                  |
| Unit            | Measurement unit                |
| SourceID        | Supporting source               |
| Confidence      | Confidence level                |
| LastUpdated     | Date of last verification       |

---

## Notes

Every Technical record belongs to either:

* one Vehicle; or
* one Configuration.

Vehicle-level Technical records describe characteristics shared across all Configurations.

Configuration-level Technical records describe trim-specific differences.

Only one of VehicleID or ConfigurationID should be populated for a single Technical record.

Technical data should remain factual and independently verifiable.

---

# 05_Equipment

## Purpose

Represents the availability of equipment for a specific Configuration.

Equipment answers:

> Does this Configuration include this feature?

Equipment never defines the feature itself.

---

## Worksheet Type

Operational

---

## Primary Key

EquipmentID

---

## References

* ConfigurationID
* EquipmentDefinitionID
* SourceID

---

## Referenced By

* 08_Evidence

---

## Columns

| Column                | Description                        |
| --------------------- | ---------------------------------- |
| EquipmentID           | Stable equipment record identifier |
| ConfigurationID       | Parent configuration               |
| EquipmentDefinitionID | Equipment definition               |
| Availability          | Availability status                |
| SourceID              | Supporting source                  |
| Confidence            | Confidence level                   |

---

## Notes

Equipment describes availability only.

Feature definitions belong exclusively to **06_EquipmentDefinitions**.

Each Equipment record references exactly one Configuration and one EquipmentDefinition.

Availability values shall follow the framework enumerations.

---

# 06_EquipmentDefinitions

## Purpose

Defines reusable equipment concepts.

Equipment Definitions answer:

> What is this feature?

They never describe whether the feature is available.

---

## Worksheet Type

Reference

---

## Primary Key

EquipmentDefinitionID

---

## References

None

---

## Referenced By

* 05_Equipment

---

## Columns

| Column                | Description                                          |
| --------------------- | ---------------------------------------------------- |
| EquipmentDefinitionID | Stable equipment definition identifier               |
| Category              | Equipment category                                   |
| Name                  | Feature name                                         |
| Description           | Feature description                                  |
| Weighted              | Indicates whether the feature contributes to scoring |
| Notes                 | Optional framework notes                             |

---

## Notes

Each Equipment Definition exists exactly once.

Equipment Definitions should remain stable across framework versions whenever possible.

The **Category** column represents a domain classification rather than a framework enumeration.

Category values follow the framework's naming conventions but are intentionally excluded from the enumeration contract.

Introducing a new Equipment Category does not require changes to `framework/architecture/enumerations.md`.

---

# 07_Reviews

## Purpose

Represents qualitative interpretation of documented observations.

Reviews answer:

> What does the evidence mean?

Reviews never contain raw observations.

---

## Worksheet Type

Operational

---

## Primary Key

ReviewID

---

## References

* VehicleID or ConfigurationID
* EvidenceID
* FrameworkVersion

---

## Referenced By

* 10_Scoring

---

## Columns

| Column           | Description                     |
| ---------------- | ------------------------------- |
| ReviewID         | Stable review identifier        |
| VehicleID        | Parent vehicle (optional)       |
| ConfigurationID  | Parent configuration (optional) |
| Category         | Review category                 |
| Score            | Review assessment               |
| Confidence       | Confidence level                |
| Summary          | Human-readable interpretation   |
| EvidenceID       | One or more supporting Evidence identifiers, comma-separated |
| FrameworkVersion | Framework version used          |

---

## Notes

Reviews interpret Evidence.

Reviews never replace Evidence.

Every Review shall reference at least one Evidence record.

Only one of VehicleID or ConfigurationID should be populated for a single Review.

Vehicle Reviews describe characteristics shared across all Configurations.

Configuration Reviews describe purchasable differences.

The **Category** column represents a framework-defined domain classification.

Review Categories are intentionally excluded from the framework enumeration contract.

New Review Categories may be introduced without modifying `framework/architecture/enumerations.md`.

Where a Review references more than one Evidence record, `EvidenceID` shall contain a comma-separated list of Evidence identifiers. Every identifier in that list shall resolve to an existing `EvidenceID` (see ADR-006).

---

# 08_Evidence

## Purpose

Represents documented observations.

Evidence answers:

> What do we know?

Evidence never interprets observations.

Evidence never assigns scores.

---

## Worksheet Type

Operational

---

## Primary Key

EvidenceID

---

## References

* VehicleID or ConfigurationID
* SourceID

---

## Referenced By

* 07_Reviews

---

## Columns

| Column          | Description                     |
| --------------- | ------------------------------- |
| EvidenceID      | Stable evidence identifier      |
| VehicleID       | Parent vehicle (optional)       |
| ConfigurationID | Parent configuration (optional) |
| Observation     | Documented observation          |
| SourceID        | Supporting source               |
| Confidence      | Confidence level                |

---

## Notes

Evidence should remain factual.

Evidence should be independently verifiable.

Only one of VehicleID or ConfigurationID should be populated for a single Evidence record.

Evidence supports Reviews but is never modified by them.

Evidence shall never reference a Review. The Reference Workbook previously exposed a `ReviewID` column on this worksheet; this diverged from the documented schema above and has been removed (ADR-006).

The **Evidence Type** enumeration (`framework/architecture/enumerations.md`) is defined but intentionally not yet represented as a workbook column. This is a known, deferred gap, not an oversight.

---

# 09_Sources

## Purpose

Represents the origin of information.

Sources answer:

> Where did this information come from?

Sources never interpret information.

---

## Worksheet Type

Reference

---

## Primary Key

SourceID

---

## References

None

---

## Referenced By

* 04_Technical
* 05_Equipment
* 08_Evidence

---

## Columns

| Column        | Description              |
| ------------- | ------------------------ |
| SourceID      | Stable source identifier |
| Type          | Source type              |
| Title         | Source title             |
| Publisher     | Publisher                |
| URL           | Original location        |
| PublishedDate | Publication date         |
| RetrievedDate | Retrieval date           |
| Notes         | Optional notes           |

---

## Notes

Each Source should exist only once.

Evidence, Technical and Equipment reference Sources.

Reviews never reference Sources directly.

---

# 10_Scoring

## Purpose

Stores calculated framework evaluations.

Scores are generated.

They are never entered manually.

---

## Worksheet Type

Calculated

---

## Primary Key

ScoreID

---

## References

* ConfigurationID
* CriterionID
* ReviewID
* FrameworkVersion

---

## Referenced By

None

---

## Columns

| Column           | Description                |
| ---------------- | -------------------------- |
| ScoreID          | Stable score identifier    |
| ConfigurationID  | Evaluated configuration    |
| CriterionID      | Evaluated criterion        |
| ReviewID         | Supporting review          |
| RawScore         | Normalized criterion score |
| WeightedScore    | Weighted criterion score   |
| Explanation      | Human-readable explanation |
| FrameworkVersion | Framework version used     |

---

## Notes

Scores belong to Configurations.

Vehicle-level Overall Scores shall not exist.

Every Score shall be reproducible from:

* Review;
* Criterion;
* FrameworkVersion.

Scores should never be manually edited.

`RawScore` and `WeightedScore` shall be implemented as formulas that look up `Review.Score` (07_Reviews) and `Criterion.Weight` (01_Criteria) by identifier, not as manually entered or pasted values (ADR-005).

`RawScore = ROUND(Score / 5 * 100, 2)`. `WeightedScore = ROUND(RawScore * Weight / 100, 2)`. `Review.Score` uses a fixed 1–5 integer scale (see `docs/03_scoring-model.md`).

Configuration-level Overall Scores are calculated in `12_OverallScores`, not in this worksheet.

---

# 11_DecisionLog

## Purpose

Maintains the architectural and implementation history of the Reference Workbook.

DecisionLog documents significant framework and implementation changes.

---

## Worksheet Type

Metadata

---

## Primary Key

DecisionID

---

## References

* FrameworkVersion

---

## Referenced By

None

---

## Columns

| Column           | Description                          |
| ---------------- | ------------------------------------ |
| DecisionID       | Stable decision identifier           |
| Date             | Decision date                        |
| FrameworkVersion | Applicable framework version         |
| Category         | Decision category                    |
| Decision         | Decision summary                     |
| Rationale        | Reason for the decision              |
| Reference        | ADR, Issue or Pull Request reference |

---

## Notes

DecisionLog documents implementation history.

Architectural decisions should reference the corresponding ADR whenever applicable.

DecisionLog improves traceability between documentation, implementation and project history.

---

# 12_OverallScores

## Purpose

Stores the aggregated Overall Score for one Configuration under one Framework Version.

Overall Scores are generated.

They are never entered manually.

They introduce no new information beyond what is already recorded in `10_Scoring`.

---

## Worksheet Type

Calculated

---

## Primary Key

OverallScoreID

---

## References

* ConfigurationID
* FrameworkVersion

---

## Referenced By

None

---

## Columns

| Column           | Description                                                            |
| ---------------- | ------------------------------------------------------------------------ |
| OverallScoreID   | Stable overall score identifier                                          |
| ConfigurationID  | Evaluated configuration                                                  |
| FrameworkVersion | Framework version used                                                   |
| OverallScore     | Sum of WeightedScore across this Configuration's 10_Scoring rows         |
| CoveragePercent  | Percentage of total WEIGHTED + Active Criterion weight actually scored   |
| CriteriaScored   | Count of WEIGHTED + Active criteria with a Score for this Configuration  |
| CriteriaTotal    | Count of all WEIGHTED + Active criteria in 01_Criteria                   |
| Notes            | Optional notes, e.g. flagging incomplete coverage                        |

---

## Notes

An OverallScore shall never be interpreted, displayed, or exported without its `CoveragePercent` alongside it.

`CoveragePercent` below 100 marks the Overall Score as a partial, not a complete, evaluation. It is a real and reproducible number, never a substitute for the missing criteria.

OverallScore does not reference a single Review or a single Score row. It aggregates all `10_Scoring` rows for the given `ConfigurationID` and `FrameworkVersion` (ADR-005).

OverallScore should never be manually edited.

---

# Relationship Mapping

This section describes how the logical relationships defined by the framework are represented within the Reference Workbook.

The workbook implements relationships using stable identifiers.

Worksheet order does not define relationships.

Relationships are always explicit.

---

## Vehicle

```text id="kqhygn"
VehicleID
        │
        ▼
Configuration
```

One Vehicle may own multiple Configurations.

Every Configuration shall reference exactly one Vehicle.

---

## Technical

```text id="c3eq4x"
Vehicle
        │
        ├──────────────┐
        ▼              ▼
Technical        Configuration
```

Technical records belong to either:

* one Vehicle; or
* one Configuration.

Both identifiers shall never be populated simultaneously.

---

## Equipment

```text id="93wq74"
Configuration
        │
        ▼
Equipment
        │
        ▼
EquipmentDefinition
```

Equipment represents feature availability.

Equipment Definitions represent reusable feature concepts.

---

## Evidence

```text id="dzvj3o"
Vehicle or Configuration
            │
            ▼
        Evidence
            │
            ▼
         Source
```

Evidence documents observations.

Evidence always references at least one Source.

---

## Review

```text id="k7jzq7"
Vehicle or Configuration
            │
            ▼
         Review
            │
            ▼
        Evidence
```

Reviews interpret documented Evidence.

Reviews never reference Sources directly.

---

## Scoring

```text id="f0p5b9"
Configuration
        │
        ▼
Criterion
        │
        ▼
Review
        │
        ▼
FrameworkVersion
        │
        ▼
Score
```

Scores represent calculated framework evaluations.

Scores never own source information.

---

# Validation Rules

Every implementation of the Reference Workbook shall validate the following rules before scoring.

---

## Identifier Validation

Every identifier shall be:

* unique;
* stable;
* non-empty.

Duplicate identifiers are implementation errors.

---

## Relationship Validation

Every Configuration shall reference an existing Vehicle.

Every Equipment shall reference:

* an existing Configuration;
* an existing EquipmentDefinition.

Every Evidence shall reference:

* an existing Vehicle or Configuration;
* an existing Source.

Every Review shall reference:

* an existing Vehicle or Configuration;
* at least one existing Evidence record.

Where `EvidenceID` contains a comma-separated list, every individual identifier in that list shall resolve to an existing `EvidenceID` (ADR-006).

Every Score shall reference:

* an existing Configuration;
* an existing Criterion;
* an existing Review;
* an existing FrameworkVersion.

Broken references invalidate framework compliance.

---

## Enumeration Validation

Only worksheet columns explicitly backed by framework enumerations shall use enumeration validation.

Enumeration values shall conform to:

framework/architecture/enumerations.md

Unknown enumeration values shall be treated as validation errors.

Columns representing domain classifications rather than framework enumerations shall follow documented naming conventions but shall not be constrained by the enumeration contract.

---

## Ownership Validation

Information shall exist in only one worksheet.

Examples

Vehicle dimensions shall exist only in:

```text id="eon6c4"
04_Technical
```

Equipment definitions shall exist only in:

```text id="ekn9cm"
06_EquipmentDefinitions
```

Reviews shall never duplicate Evidence.

Evidence shall never duplicate Source metadata.

---

## Version Validation

Every calculated Score shall reference a valid FrameworkVersion.

Historical evaluations shall retain their original FrameworkVersion.

Framework versions shall never be overwritten.

### Exclusive Parent Rule

Some entities may belong to either a Vehicle or a Configuration.

Where both `VehicleID` and `ConfigurationID` are present, exactly one shall be populated.

Populating both identifiers or leaving both empty is a validation error.

This rule preserves unambiguous ownership throughout the framework.

---

# Calculation Boundaries

The workbook intentionally separates editable information from calculated information.

Only the following worksheets contain calculated framework evaluations.

```text id="xyyu0i"
10_Scoring
12_OverallScores
```

Operational worksheets shall never contain calculated scores.

Operational worksheets may contain helper formulas for validation or lookup provided they do not change framework behaviour.

---

## Editable Worksheets

The following worksheets contain manually maintained information.

* README
* 01_Criteria
* 02_Vehicles
* 03_Configurations
* 04_Technical
* 05_Equipment
* 06_EquipmentDefinitions
* 07_Reviews
* 08_Evidence
* 09_Sources
* 11_DecisionLog

---

## Calculated Worksheets

```text id="l7nknw"
10_Scoring
12_OverallScores
```

Scores shall always be derived from documented framework rules.

Manual modification of calculated scores is prohibited.

---

# Data Entry Rules

When entering information into the workbook:

1. Always reference an existing VehicleID or ConfigurationID as appropriate.
2. Always reference an existing SourceID whenever factual information is recorded.
3. Never duplicate information owned by another worksheet.
4. Unknown is preferable to assumed.
5. Confidence should be recorded whenever qualitative judgement is involved.
6. Preserve traceability for every Review and Score.
7. Never overwrite historical observations without recording a newer observation.
8. Notes are intended for implementation comments, clarification and data collection context.

Notes are never evaluated and never contribute to scoring.

---

# Naming Conventions

Identifiers are immutable.

Once assigned, an identifier shall never be reused or modified.

Identifiers should remain stable across framework versions.

Examples

VehicleID

```text id="sb7khi"
KIA_EV2
RENAULT_4
SKODA_ELROQ
```

ConfigurationID

```text id="2x8xv2"
CONF_KIA_EV2_EXCLUSIVE_NO
CONF_KIA_EV2_GTLINE_NO
```

TechnicalID

```text id="ylmjxb"
TECH_000001
```

EquipmentDefinitionID

```text id="yywdju"
EQDEF_360_CAMERA
EQDEF_MATRIX_LED
EQDEF_OTA
```

EquipmentID

```text id="xlyte9"
EQ_000001
```

EvidenceID

```text id="fhf13o"
EV_000001
```

ReviewID

```text id="wjzvk5"
REV_000001
```

CriterionID

```text id="3hk18m"
CR_DRIVING
CR_TECHNOLOGY
CR_VALUE
```

ScoreID

```text id="gjrr1s"
SCORE_000001
```

OverallScoreID

```text id="ovsc01"
OVSC_000001
```

Identifiers should never encode mutable information.

---

# Migration Principles

The workbook is intentionally structured to support migration without redesign.

Expected migration targets include:

* Google Sheets;
* SQLite;
* PostgreSQL;
* REST APIs;
* GraphQL;
* Web applications.

Future implementations should preserve:

* entity ownership;
* identifier stability;
* framework behaviour;
* traceability;
* reproducibility.

Technology may change.

Framework methodology shall remain unchanged.

---

# Workbook Compliance

A workbook is considered framework compliant only if it satisfies all of the following:

* implements the documented workbook structure;
* preserves entity ownership;
* maintains referential integrity;
* preserves framework traceability;
* produces deterministic scoring;
* references FrameworkVersion for every evaluation;
* remains consistent with all accepted ADRs.

Passing spreadsheet validation alone is not sufficient for framework compliance.

---

# Reference Implementation

The workbook is the official Reference Implementation of the EV Decision Framework.

Future implementations may choose different technologies.

They shall implement the same framework methodology without changing framework behaviour.

---

# Workbook Invariants

The following rules are fundamental to every compliant Reference Workbook.

These rules shall never be violated.

- A Configuration shall reference exactly one Vehicle.
- A Technical record shall reference either one Vehicle or one Configuration.
- An Equipment record shall reference exactly one Configuration.
- An Equipment record shall reference exactly one EquipmentDefinition.
- An Evidence record shall reference at least one Source.
- A Review shall reference at least one Evidence record.
- A Score shall reference exactly one Configuration.
- A Score shall reference exactly one Criterion.
- A Score shall reference exactly one Review.
- An OverallScore shall reference exactly one Configuration.
- An OverallScore shall reference exactly one FrameworkVersion.
- An OverallScore shall never reference a single Review or a single Score.
- An OverallScore shall always carry a CoveragePercent.
- EquipmentDefinitions define features.
- Equipment records define availability.
- Reviews interpret Evidence.
- Evidence documents observations.
- Sources provide information.

---

# Guiding Principle

> **The Reference Workbook is a faithful implementation of the framework architecture. It preserves ownership, traceability and reproducibility while remaining simple enough to understand, maintain and migrate.**
