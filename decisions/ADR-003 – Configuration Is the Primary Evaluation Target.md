# ADR-003 – Configuration Is the Primary Evaluation Target

**Status:** Accepted
**Date:** 2026-07-04
**Framework Version:** 1.0

---

# Context

The EV Decision Framework exists to support purchasing decisions.

During the initial framework design, the Vehicle was assumed to be the primary evaluation target.

However, architectural review identified a mismatch between the framework model and real purchasing behaviour.

Customers do not purchase a vehicle model in the abstract.

They purchase a specific market configuration.

Examples include:

* Kia EV2 Exclusive
* Kia EV2 GT-Line
* Renault 4 Techno
* Renault 4 Iconic

Configurations may differ in:

* price;
* equipment;
* software features;
* driver assistance systems;
* interior quality;
* market availability.

These differences directly influence purchasing decisions.

---

# Problem

Treating Vehicle as the only evaluated entity introduces several problems.

Different configurations would receive identical scores despite having different:

* equipment;
* purchase price;
* feature availability;
* ownership value.

Conversely, treating Configuration as the only meaningful entity would duplicate large amounts of shared information.

Examples include:

* platform characteristics;
* driving dynamics;
* crash safety;
* cabin refinement;
* battery chemistry;
* vehicle dimensions.

These characteristics generally belong to the Vehicle rather than individual configurations.

---

# Decision

The framework distinguishes between:

## Vehicle

Represents the common product.

Vehicle owns characteristics shared by every configuration.

## Configuration

Represents the purchasable product.

Configuration is the primary evaluation target for purchasing decisions.

All rankings, comparisons and purchase recommendations shall therefore be performed at the Configuration level.

---

# Responsibilities

## Vehicle

Vehicle owns information that is independent of trim level.

Examples include:

* manufacturer;
* model;
* platform;
* dimensions;
* battery architecture;
* vehicle-wide technical characteristics;
* shared evidence;
* shared reviews.

Vehicle acts as the foundation from which configurations inherit common information.

---

## Configuration

Configuration owns information that differs between purchasable variants.

Examples include:

* trim;
* market;
* pricing;
* equipment;
* option packages;
* configuration-specific software;
* configuration-specific evidence;
* configuration-specific reviews.

Configuration represents the actual purchasing choice.

---

# Evaluation Model

The framework evaluates both entities for different purposes.

Vehicle contributes common knowledge.

Configuration produces purchasing decisions.

The evaluation pipeline therefore becomes:

```text id="z1rj9v"
Vehicle
    │
    ▼
Configuration
    │
    ├── Vehicle Information
    ├── Configuration Information
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
    │
    ▼
Recommendation
```

Vehicle-level information should be reused whenever possible.

Configuration-specific information should only be introduced where meaningful differences exist.

---

# Scoring Rules

Vehicle may contribute to evaluation.

Configuration owns the final result.

Specifically:

* Criterion Scores belong to a Configuration.
* Overall Scores belong to a Configuration.
* Rankings belong to Configurations.
* Recommendations refer to Configurations.

Vehicle shall not receive an Overall Score.

Vehicle-level assessments may exist only as reusable inputs to configuration-level evaluations.

---

# Technical Data

Technical data may belong to either entity.

Vehicle-level examples:

* dimensions;
* platform;
* wheelbase;
* battery chemistry;
* crash structure.

Configuration-level examples:

* battery capacity (where applicable);
* charging capability;
* wheel size;
* option-dependent specifications.

The framework should avoid duplication whenever information is identical across configurations.

---

# Equipment

Equipment always belongs to a Configuration.

Availability cannot be evaluated meaningfully at the Vehicle level.

---

# Evidence

Evidence may belong to either:

* Vehicle;
* Configuration.

Examples

Vehicle Evidence

* measured cabin noise;
* winter efficiency;
* suspension behaviour.

Configuration Evidence

* matrix LED performance;
* surround-view camera quality;
* premium audio system.

Evidence should always be attached to the lowest level at which it remains valid.

---

# Reviews

Reviews follow the same principle.

Vehicle Reviews

Describe characteristics common to all configurations.

Configuration Reviews

Describe characteristics introduced by the selected trim or options.

Configuration reviews may reference both:

* Vehicle Evidence;
* Configuration Evidence.

---

# Benefits

This model provides several advantages.

## Purchasing Accuracy

Recommendations reflect products that customers can actually purchase.

---

## Reduced Duplication

Shared information is stored once.

Configuration-specific information is stored only where necessary.

---

## Better Maintainability

Changes affecting every configuration require modification in only one place.

---

## Improved Scalability

The framework naturally supports:

* market-specific trims;
* option packages;
* limited editions;
* future model revisions.

without redesign.

---

# Alternatives Considered

## Vehicle as the Only Evaluation Target

Rejected.

Fails to distinguish between purchasable products.

Produces misleading rankings.

---

## Configuration as the Only Meaningful Entity

Rejected.

Duplicates shared information.

Weakens maintainability.

Makes evidence reuse unnecessarily difficult.

---

## Evaluate Both Equally

Rejected.

Creates ambiguity regarding which score should drive recommendations.

The framework requires a single purchasing target.

Configuration fulfils that role.

---

# Consequences

Future implementations should:

* store shared knowledge at the Vehicle level;
* store purchasing differences at the Configuration level;
* generate all rankings from Configuration scores.

Vehicle remains an essential framework entity but is no longer the primary decision output.

---

# Related Documents

* Entity Model
* Relationships Model
* Workbook Schema
* Data Flow
* Implementation Contract

---

# Guiding Principle

> **The framework stores knowledge at the highest level where it remains true, but always evaluates and recommends the product that can actually be purchased.**
