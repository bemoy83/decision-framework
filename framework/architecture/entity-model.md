# EV Decision Framework

## Entity Model

**Version:** 1.2
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the core entities used by the EV Decision Framework.

Entities represent the fundamental building blocks of the framework.

Each entity has:

* a unique identity;
* clearly defined ownership;
* explicit relationships;
* and a single responsibility.

The entity model is implementation-independent and applies equally to spreadsheets, databases and future software implementations.

---

# Design Principles

The entity model follows six principles.

1. Every entity has one responsibility.
2. Every entity has one identity.
3. Every entity owns only the information it creates.
4. Relationships between entities are explicit.
5. Data duplication should be minimized.
6. Entities remain independent of implementation technology.

---

# Ownership

The framework distinguishes between ownership and references.

An entity owns only the information it creates.

An entity may reference information owned by another entity.

Ownership determines where information belongs.

References determine how information is connected.

---

# Core Entities

```text id="z6mxik"
Vehicle
Configuration
Technical
Criterion
EquipmentDefinition
Equipment
Evidence
Source
Review
Score
FrameworkVersion
Decision
```

---

# Vehicle

## Purpose

Represents the shared identity of a vehicle model.

Examples

* Kia EV2
* Renault 4
* Škoda Elroq

Identity

VehicleID

Owns

* manufacturer
* model
* platform
* lifecycle status
* shared characteristics

Vehicle may also own:

* shared Technical records;
* shared Evidence;
* shared Reviews.

Vehicle does **not** represent a purchasable product.

Vehicle groups one or more Configurations.

---

# Configuration

## Purpose

Represents a purchasable vehicle configuration.

Examples

* Exclusive
* GT-Line
* Techno

Identity

ConfigurationID

Belongs to

Vehicle

Owns

* trim
* market
* status
* pricing
* package information
* configuration-specific characteristics

Configuration Status represents the commercial lifecycle of the purchasable Configuration and is independent of Vehicle Status.

Configuration is the primary evaluation target of the framework.

All rankings, comparisons and purchase recommendations refer to Configurations.

---

# Technical

## Purpose

Represents one measurable technical characteristic.

Examples

* Overall Length
* Wheelbase
* Battery Capacity
* Maximum DC Charging Power
* WLTP Range
* Kerb Weight

Identity

TechnicalID

Owns

* property
* value
* unit
* confidence
* source reference
* last updated

Technical represents verified factual information.

Technical never contains interpretation or evaluation.

Each Technical record belongs to either:

* one Vehicle; or
* one Configuration,

depending on where the characteristic remains true.

---

# Criterion

## Purpose

Represents one evaluation criterion.

Examples

* Driving Experience
* Technology
* Long-Term Ownership

Identity

CriterionID

Owns

* name
* category
* description
* weighting
* requirement type

Criteria define **what** is evaluated.

They never store evaluation results.

---

# EquipmentDefinition

## Purpose

Represents a reusable equipment concept.

Examples

* 360 Camera
* Matrix LED Headlights
* Heated Steering Wheel
* OTA Updates
* Adaptive Cruise Control

Identity

EquipmentDefinitionID

Owns

* name
* category
* description
* weighted status

EquipmentDefinition describes **what a feature is**.

It never describes availability.

---

# Equipment

## Purpose

Represents the availability of an EquipmentDefinition for one Configuration.

Identity

EquipmentID

Owns

* availability
* confidence
* source reference

References

* Configuration
* EquipmentDefinition

Equipment answers:

> Does this Configuration include this feature?

Equipment never defines the feature itself.

---

# Source

## Purpose

Represents the origin of information.

Examples

* Manufacturer
* Motor.no
* Euro NCAP
* Independent reviewers

Identity

SourceID

Owns

* publisher
* publication
* URL
* publication date
* retrieval date

Sources provide information.

Sources never interpret information.

---

# Evidence

## Purpose

Represents a documented observation.

Examples

* Measured cabin noise
* Verified charging curve
* Independent winter range
* OTA update history

Identity

EvidenceID

Owns

* observation
* confidence
* supporting source

References

* Vehicle or Configuration
* Source

Evidence answers:

> What do we know?

Evidence never contains interpretation.

Evidence never assigns scores.

---

# Review

## Purpose

Represents a qualitative interpretation of Evidence.

Examples

* Ride comfort
* Software quality
* Steering precision
* Cabin refinement

Identity

ReviewID

Owns

* category
* summary
* confidence

References

* Vehicle or Configuration
* Evidence

Review answers:

> What does the evidence mean?

Reviews interpret Evidence.

Reviews never modify Evidence.

---

# Score

## Purpose

Represents one calculated framework evaluation.

Identity

ScoreID

Owns

* criterion score
* weighted score
* explanation

References

* Configuration
* Review
* FrameworkVersion

Scores are generated.

Scores are never manually entered.

---

# FrameworkVersion

## Purpose

Represents one released framework definition.

Examples

* 1.0
* 1.1
* 2.0

Identity

FrameworkVersion

Owns

* methodology version
* weighting version
* schema version

Framework Versions guarantee reproducibility.

Historical versions are immutable.

---

# Decision

## Purpose

Represents one documented architectural decision.

Examples

* ADR acceptance
* methodology changes
* framework revisions

Identity

DecisionID

Owns

* description
* rationale
* date
* framework version

Decisions document why the framework evolved.

They do not implement changes themselves.

---

# Entity Responsibilities

Each entity owns only the information it creates.

| Entity              | Responsibility        |
| ------------------- | --------------------- |
| Vehicle             | Shared identity       |
| Configuration       | Purchasable product   |
| Technical           | Measurable facts      |
| EquipmentDefinition | Feature definition    |
| Equipment           | Feature availability  |
| Source              | Information origin    |
| Evidence            | Verified observations |
| Review              | Interpretation        |
| Criterion           | Evaluation definition |
| Score               | Calculated evaluation |
| FrameworkVersion    | Framework identity    |
| Decision            | Architectural history |

---

# Entity Lifecycle

Entities evolve independently.

Examples

Vehicle

May gain new Configurations.

Configuration

May gain new Equipment.

Evidence

May gain stronger supporting Sources.

Review

May evolve as new Evidence becomes available.

FrameworkVersion

Never changes after release.

---

# Entity Independence

The entity model supports implementations using:

* Excel
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* GraphQL
* Web applications

without changing entity definitions.

---

# Guiding Principle

> **Entities own knowledge. Relationships connect knowledge. Reviews interpret knowledge. Scores evaluate interpretations.**
