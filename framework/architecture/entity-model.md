# EV Decision Framework

## Entity Model

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines the core entities used by the EV Decision Framework.

Entities represent the fundamental building blocks of the framework.

Each entity has:

* a unique identity,
* clearly defined ownership,
* relationships to other entities,
* and a specific responsibility.

The entity model is independent of any implementation technology.

---

# Design Principles

The entity model follows five principles.

1. Every entity has one clear responsibility.
2. Every entity has a unique identifier.
3. Data should exist in only one place.
4. Relationships should be explicit.
5. Entities should remain reusable across implementations.

---

# Core Entities

The framework currently defines the following entities.

```text
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

Represents a physical vehicle model.

Examples

* Kia EV2
* Renault 4
* Škoda Elroq

Identity

VehicleID

Owns

* identity
* manufacturer
* model
* platform
* lifecycle status

Does NOT own

* equipment
* reviews
* technical specifications
* scores

---

# Configuration

Represents a purchasable vehicle configuration.

Examples

* Exclusive
* GT-Line
* Tech Pack

Identity

ConfigurationID

Belongs to

Vehicle

Owns

* trim
* market
* pricing
* package information

---

# Technical

Represents a single measurable technical characteristic of a Vehicle or Configuration.

Examples

* Overall Length
* Wheelbase
* Battery Gross Capacity
* Battery Net Capacity
* Maximum DC Charging Power
* WLTP Range
* Kerb Weight

Identity

TechnicalID

Owns

* Property
* Value
* Unit
* Source
* Confidence
* Last Updated

Technical represents verified factual information.

Technical data never contains interpretation, evaluation or recommendations.

Each Technical record belongs to either:

* one Vehicle; or
* one Configuration,

depending on the scope of the characteristic.

Whenever a specification applies to every configuration, it should belong to the Vehicle.

Configuration-specific specifications should only be stored at the Configuration level.

---

# Criterion

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

---

# EquipmentDefinition

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

* Name
* Category
* Description
* Weighted Status

Equipment Definitions describe *what* a feature is.

They do not describe whether a specific Configuration includes that feature.

Availability is managed by the Equipment entity.

---

# Equipment

Represents the availability of an Equipment Definition for a specific Configuration.

Examples

* 360 Camera
* Matrix LED
* OTA Updates
* Ambient Lighting

Identity

EquipmentID

Owns

- Availability
- Configuration
- Confidence
- Source

Equipment references exactly one EquipmentDefinition.

---

# Evidence

Represents a single factual observation supporting a claim.

Examples

Measured cabin noise

Verified charging curve

Independent winter range

Identity

EvidenceID

Owns

* claim
* evidence
* confidence
* supporting source

Evidence never contains conclusions.

---

# Review

Represents a qualitative assessment.

Examples

Software usability

Ride comfort

Steering precision

Identity

ReviewID

Owns

* category
* summary
* score
* confidence

Reviews interpret evidence.

They do not replace it.

---

# Source

Represents one information source.

Examples

Manufacturer

Motor.no

Euro NCAP

Identity

SourceID

Owns

* publisher
* publication
* URL
* publication date
* retrieval date

Sources never evaluate.

They only provide information.

---

# Score

Represents one calculated framework result.

Identity

ScoreID

Owns

* criterion
* raw score
* weighted score
* explanation

Scores are generated.

They are never manually entered.

---

# FrameworkVersion

Represents one released framework definition.

Examples

1.0

1.1

2.0

Owns

* methodology version
* weighting version
* schema version

Vehicle evaluations always reference one framework version.

---

# Decision

Represents a documented project decision.

Examples

Framework changes

Weighting updates

Methodology revisions

ADR references

Identity

DecisionID

Owns

* description
* rationale
* date
* framework version

---

# Entity Ownership

Each entity owns only its own data.

Example

Vehicle owns:

* manufacturer
* model

Vehicle does NOT own:

* WLTP
* Reviews
* Equipment
* Sources

Those belong to their respective entities.

---

# Entity Lifecycle

Entities evolve independently.

Examples

Vehicle

May receive new reviews.

Review

May receive stronger evidence.

Evidence

May receive additional supporting sources.

FrameworkVersion

Never changes after release.

---

# Entity Independence

The framework should allow future implementations using:

* Excel
* SQLite
* PostgreSQL
* REST APIs
* GraphQL
* Web applications

without changing entity definitions.

---

# Guiding Principle

Entities represent knowledge.

Relationships connect knowledge.

The framework evaluates relationships rather than individual entities in isolation.
