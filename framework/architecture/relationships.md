# EV Decision Framework

## Relationships Model

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines the logical relationships between entities in the EV Decision Framework.

Relationships describe how entities interact while maintaining data integrity, traceability and explainability.

The model is implementation-independent and applies equally to spreadsheets, relational databases and future APIs.

---

# Design Principles

Relationships should follow these principles:

* Every relationship must be explicit.
* Relationships should minimise data duplication.
* Parent entities own identity.
* Child entities reference identity.
* Relationships should remain stable across framework versions.

---

# Entity Relationship Overview

```text
FrameworkVersion
        │
        │
        ▼
Criteria ───────────────┐
                        │
                        ▼
                     Score
                        ▲
                        │
Vehicle ────────────────┼──────────────┐
│                       │              │
│                       │              │
▼                       ▼              ▼
Configuration      Review         Technical
                        ▲
                        │
                        ▼
                    Evidence
                        ▲
                        │
                        ▼
                     Source

Vehicle
    │
    ▼
Equipment
    │
    ▼
EquipmentDefinition

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

A vehicle may exist in multiple market configurations.

Example

KIA_EV2

* Standard
* Exclusive
* GT-Line

Each configuration belongs to exactly one vehicle.

---

## Vehicle → Technical

Relationship

**One-to-Many (1:N)**

Each vehicle has multiple technical attributes.

Examples

* Length
* Battery
* WLTP
* Charging speed

Each technical record belongs to one vehicle.

---

## Vehicle → Equipment

Relationship

**One-to-Many (1:N)**

Each vehicle contains multiple equipment records.

Availability is stored per configuration where applicable.

---

## Equipment → EquipmentDefinition

Relationship

**Many-to-One (N:1)**

Equipment records reference a shared equipment definition.

Example

Many vehicles reference:

EQ_360_CAMERA

The definition exists only once.

---

## Vehicle → Review

Relationship

**One-to-Many (1:N)**

A vehicle may receive multiple reviews.

Examples

* Software
* Ride Comfort
* Steering
* Cabin Noise

Each review belongs to one vehicle.

---

## Review → Evidence

Relationship

**One-to-Many (1:N)**

A review should be supported by one or more evidence records.

Evidence strengthens confidence.

Reviews summarise evidence.

---

## Evidence → Source

Relationship

**Many-to-One (N:1)**

Multiple evidence records may originate from the same source.

Example

Motor.no

↓

Cabin noise

↓

Charging curve

↓

Software observations

The source exists only once.

---

## Technical → Source

Relationship

**Many-to-One (N:1)**

Technical data should reference the source used to verify the specification.

---

## Equipment → Source

Relationship

**Many-to-One (N:1)**

Equipment availability should reference the official source whenever possible.

---

## Score → Criterion

Relationship

**Many-to-One (N:1)**

Every calculated score references exactly one criterion.

---

## Score → Vehicle

Relationship

**Many-to-One (N:1)**

Each vehicle receives one score per criterion.

The total score is derived from these individual criterion scores.

---

## FrameworkVersion → Criteria

Relationship

**One-to-Many (1:N)**

Each framework version defines one complete criteria model.

Historical framework versions remain immutable.

---

## FrameworkVersion → Score

Relationship

**One-to-Many (1:N)**

Every calculated score references the framework version used during evaluation.

Historical evaluations remain reproducible.

---

## Decision → FrameworkVersion

Relationship

**Many-to-One (N:1)**

Every architecture decision references the framework version that introduced it.

---

# Relationship Ownership

Ownership defines where information originates.

| Parent           | Child         |
| ---------------- | ------------- |
| Vehicle          | Configuration |
| Vehicle          | Technical     |
| Vehicle          | Equipment     |
| Vehicle          | Review        |
| Review           | Evidence      |
| Source           | Evidence      |
| Criterion        | Score         |
| FrameworkVersion | Criteria      |

Parents own identity.

Children reference identity.

---

# Referential Integrity

The framework should enforce the following rules.

## Mandatory Relationships

A Review:

* must reference a Vehicle.

An Evidence record:

* must reference a Review.
* must reference at least one Source.

A Score:

* must reference a Vehicle.
* must reference a Criterion.
* must reference a FrameworkVersion.

A Technical record:

* must reference a Vehicle.

---

## Optional Relationships

Some entities may legitimately exist without children.

Examples

A newly added vehicle may temporarily have:

* no reviews
* no scores
* incomplete equipment

The framework should represent missing information as incomplete rather than incorrect.

---

# Relationship Constraints

The framework intentionally avoids circular dependencies.

Information should always flow in one direction.

```text
Source
    ↓
Evidence
    ↓
Review
    ↓
Score
```

Scores must never modify reviews.

Reviews must never modify evidence.

Evidence must never modify source data.

---

# Traceability

Every recommendation should be traceable.

Example

Final Score

↓

Criterion

↓

Review

↓

Evidence

↓

Source

This chain allows every conclusion to be explained.

---

# Future Compatibility

The relationship model has been designed to support:

* Excel
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* GraphQL
* Web applications

without modification.

---

# Guiding Principle

Relationships create meaning.

Individual entities store information.

The framework derives knowledge by connecting entities through explicit, traceable relationships.
