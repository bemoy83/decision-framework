# EV Decision Framework

## Data Flow

**Version:** 1.4
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines how information flows through the EV Decision Framework.

The framework transforms raw information into explainable, traceable and reproducible purchase recommendations.

The data flow is independent of implementation technology and applies equally to spreadsheets, databases and future software implementations.

---

# Guiding Principle

The framework never evaluates raw data directly.

Information progresses through a series of increasingly meaningful stages.

Each stage adds understanding while preserving traceability to the previous stage.

Every stage has one responsibility.

---

# High-Level Flow

```text id="efdjlwm"
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
               │
               ▼
        Recommendation
```

Every stage consumes information from previous stages.

No stage should bypass another unless explicitly documented.

---

# Information Sources

Before entering the framework, information originates from one or more Sources.

Examples include:

* manufacturer specifications;
* independent testing;
* owner reports;
* certification bodies;
* professional reviews.

Sources are external to the evaluation process.

They provide information but never interpretation.

---

# Stage 1 – Vehicle

## Purpose

Represents the shared identity of a vehicle model.

Examples

* Kia EV2
* Renault 4
* Škoda Elroq

Produces

* Vehicle identity
* Shared characteristics

Consumes

Nothing.

Vehicle establishes the foundation upon which Configurations are built.

---

# Stage 2 – Configuration

## Purpose

Represents a purchasable product.

Examples

* Exclusive
* GT-Line
* Techno

Produces

Configuration-specific identity.

Consumes

Vehicle.

Configuration is the framework's primary evaluation target.

All rankings and recommendations refer to Configurations.

---

# Stage 3 – Technical

## Purpose

Stores measurable facts.

Examples

* Vehicle length
* Battery capacity
* Charging speed
* Wheelbase
* Boot volume
* WLTP range

Produces

Verified technical information.

Consumes

Vehicle or Configuration.

Technical never contains interpretation.

Technical never contains recommendations.

---

# Stage 4 – Equipment

## Purpose

Describes feature availability.

Examples

* 360 Camera
* Matrix LED
* Heated Steering Wheel
* OTA Updates

Produces

Equipment availability.

Consumes

Configuration and EquipmentDefinition.

Equipment answers:

> Is this feature available?

Equipment never evaluates the usefulness of a feature.

---

# Stage 5 – Evidence

## Purpose

Transforms factual information into documented observations.

Evidence answers:

> What do we know?

Examples

* Measured cabin noise
* Verified charging curve
* Winter range measurement
* OTA update history

Evidence references one or more Sources.

Evidence never contains interpretation.

Evidence never contains evaluation.

---

# Stage 6 – Review

## Purpose

Transforms documented observations into qualitative understanding.

Review answers:

> What does the evidence mean?

Examples

* Ride comfort
* Cabin refinement
* Software quality
* Steering precision
* Visibility

Every Review references one or more Evidence records.

Evidence supports Reviews.

Reviews interpret Evidence.

Evidence never references Reviews.

---

# Stage 7 – Criterion Score

## Purpose

Transforms Reviews into framework-specific evaluations.

Each Criterion Score evaluates one Criterion.

Examples

* Driving Experience
* Technology
* Practicality
* Long-Term Ownership

Every Criterion Score references:

* one Configuration;
* one Review;
* one Framework Version.

Criterion Scores remain reproducible.

---

# Stage 8 – Overall Score

## Purpose

Aggregates Criterion Scores using the active framework weighting.

Produces

Final framework evaluation.

Overall Scores introduce no new knowledge.

They aggregate existing Criterion Scores only.

Implemented as a distinct OverallScore entity (`12_OverallScores` in the Reference Workbook), referencing Configuration and FrameworkVersion, never a single Review or Score (ADR-005).

Every Overall Score carries a coverage percentage — the proportion of the framework's weighted criteria actually scored. An Overall Score with less than full coverage is a partial, not a complete, evaluation, and must always be presented alongside its coverage figure.

---

# Stage 9 – Recommendation

## Purpose

Transforms Overall Scores into decision support.

Recommendations explain:

* strengths;
* weaknesses;
* trade-offs;
* uncertainty.

The framework provides recommendations.

The user makes the final decision.

---

# Information Flow

Information always moves forward.

```text id="uhoezjz"
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

Reverse information flow is not permitted.

Scores never modify Reviews.

Reviews never modify Evidence.

Evidence never modifies Sources.

---

# Ownership Through the Flow

Each stage owns only the information it creates.

| Stage           | Creates                 |
| --------------- | ----------------------- |
| Vehicle         | Shared identity         |
| Configuration   | Purchasable identity    |
| Technical       | Verified facts          |
| Equipment       | Feature availability    |
| Evidence        | Documented observations |
| Review          | Interpretation          |
| Criterion Score | Criterion evaluation    |
| Overall Score   | Aggregated evaluation   |
| Recommendation  | Decision support        |

Each stage references previous stages without duplicating their information.

---

# Traceability

Every recommendation must be fully explainable.

```text id="tknl1li"
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

Every conclusion should be traceable back to documented evidence and its original source.

---

# Unknown Values

Unknown information remains Unknown until supported by Evidence.

Implementations shall never replace missing information with assumptions.

Confidence should increase only as additional Evidence becomes available.

---

# Framework Versioning

Every Criterion Score and Overall Score shall reference the Framework Version used during evaluation.

Framework Version guarantees reproducibility.

Historical evaluations remain immutable.

---

# Design Constraints

The framework intentionally separates:

* Identity
* Facts
* Availability
* Observations
* Interpretation
* Evaluation
* Recommendation

This separation ensures that improvements in later stages never invalidate earlier stages.

---

# Future Compatibility

The data flow supports implementations using:

* Excel
* Google Sheets
* SQLite
* PostgreSQL
* REST APIs
* GraphQL
* AI-assisted evidence collection

without changing the conceptual model.

Future framework versions may introduce additional normalization where justified by implementation experience.

---

# Guiding Principle

> **The framework transforms facts into observations, observations into understanding, understanding into evaluation and evaluation into recommendations — while preserving complete traceability at every stage.**
