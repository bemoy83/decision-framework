# EV Decision Framework

## Data Flow

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines how information flows through the EV Decision Framework.

It describes the transformation of raw information into explainable recommendations.

The data flow is independent of implementation technology and applies equally to spreadsheets, databases and future applications.

---

# Guiding Principle

The framework does not score vehicles directly.

Instead, it progressively transforms information through a series of increasingly meaningful stages.

Each stage adds interpretation while preserving traceability to the previous stage.

---

# High-Level Flow

```text
Vehicle
    │
    ▼
Configuration
    │
    ▼
Technical Data
Equipment
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

Each stage consumes information from the previous stage.

No stage should bypass another unless explicitly documented.

---

# Stage 1 – Vehicle

Purpose

Represents the product being evaluated.

Examples

* Kia EV2
* Renault 4
* Škoda Elroq

Produces

Vehicle identity.

Consumes

Nothing.

---

# Stage 2 – Configuration

Purpose

Represents a purchasable configuration.

Examples

* Base
* Exclusive
* GT-Line

Produces

Configuration-specific identity.

Consumes

Vehicle.

---

# Stage 3 – Technical Data

Purpose

Stores measurable facts.

Examples

* Length
* Battery capacity
* Charging speed
* Boot volume
* WLTP range

Produces

Verified factual information.

Consumes

Configuration or Vehicle, depending on the specification.

Technical data never contains opinions.

---

# Stage 4 – Equipment

Purpose

Describes available equipment.

Examples

* 360 Camera
* Matrix LED
* Heated steering wheel

Produces

Equipment availability.

Consumes

Configuration.

Equipment represents availability only.

It does not evaluate usefulness.

---

# Stage 5 – Evidence

Purpose

Transforms raw information into documented observations.

Evidence should answer:

> What do we know?

Examples

* Measured cabin noise
* Verified charging curve
* Winter range test
* Software update history

Evidence should always reference one or more sources.

Evidence never assigns scores.

---

# Stage 6 – Review

Purpose

Transforms evidence into qualitative assessment.

Reviews answer:

> What does the evidence mean?

Examples

* Ride comfort
* Software quality
* Visibility
* Cabin refinement

Reviews should reference one or more Evidence records.

Multiple reviews may reuse the same evidence.

---

# Stage 7 – Criterion Score

Purpose

Transforms reviews into framework-specific scores.

Each score represents one criterion.

Examples

* Driving Experience
* Technology
* Practicality
* Long-Term Ownership

Scores should reference:

* Criterion
* Review
* Framework Version

Scores should remain reproducible.

---

# Stage 8 – Overall Score

Purpose

Aggregates criterion scores using the active weighting model.

Produces

Final framework score.

The overall score contains no new information.

It only aggregates existing criterion scores.

---

# Stage 9 – Recommendation

Purpose

Transforms the overall score into a decision-support recommendation.

The recommendation should always explain:

* strengths
* weaknesses
* trade-offs
* uncertainty

The framework recommends.

The user decides.

---

# Information Flow

Information should always move forward.

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

Information should never move backwards.

Scores must never modify reviews.

Reviews must never modify evidence.

Evidence must never modify source data.

---

# Data Ownership

Each stage owns only the information it creates.

| Stage           | Owns                     |
| --------------- | ------------------------ |
| Vehicle         | Identity                 |
| Configuration   | Market-specific identity |
| Technical Data  | Facts                    |
| Equipment       | Availability             |
| Evidence        | Verified observations    |
| Review          | Interpretation           |
| Criterion Score | Criterion evaluation     |
| Overall Score   | Aggregated evaluation    |
| Recommendation  | Decision support         |

No stage should duplicate information owned by another stage.

---

# Traceability

Every recommendation should be fully traceable.

```text
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

Every conclusion must be explainable using this chain.

---

# Unknown Values

Unknown information should remain unknown until evidence becomes available.

Unknown values must never be replaced by assumptions.

Confidence should increase as additional evidence is collected.

---

# Framework Versioning

Every Criterion Score and Overall Score shall reference the Framework Version used during evaluation.

This guarantees reproducibility across framework revisions.

---

# Design Constraints

The framework intentionally separates:

* Identity
* Facts
* Availability
* Evidence
* Interpretation
* Evaluation
* Recommendation

This separation ensures that improvements in one stage do not invalidate information in previous stages.

---

# Future Compatibility

The data flow has been designed to support:

* Spreadsheet implementations
* Relational databases
* REST APIs
* GraphQL
* Automated data pipelines
* AI-assisted evidence collection

without changing the conceptual model.

---

# Guiding Principle

The EV Decision Framework does not transform data directly into recommendations.

It transforms:

**Facts → Evidence → Understanding → Evaluation → Recommendation**

Every stage must preserve transparency, traceability and reproducibility.
