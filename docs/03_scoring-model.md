# EV Decision Framework

## Scoring Model

**Version:** 1.1
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines how the EV Decision Framework evaluates vehicle **Configurations**.

The scoring model is designed to produce recommendations that are:

* transparent;
* explainable;
* repeatable;
* reproducible.

Every Overall Score shall be reproducible using the same Framework Version and the same underlying information.

The scoring model evaluates Configurations.

Vehicle models provide shared context but are never ranked directly.

---

# Design Principles

The scoring model follows six principles.

1. Eligibility before evaluation.
2. Facts before interpretation.
3. Evidence before scoring.
4. Reviews before judgement.
5. Weighting expresses priorities.
6. Every score shall remain explainable.

---

# Evaluation Pipeline

Every Configuration follows the same evaluation pipeline.

```text id="wksr4b"
Vehicle
        │
        ▼
Configuration
        │
        ▼
Hard Requirements
        │
        ▼
Technical Facts
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

Each stage adds understanding while preserving traceability to the previous stage.

---

# Stage 1 – Configuration Eligibility

Hard Requirements determine whether a Configuration enters the evaluation pipeline.

Examples include:

* Battery electric vehicle
* Maximum vehicle length
* Norwegian market availability
* Winter motorway range requirement

Configurations failing one or more Hard Requirements shall not receive an Overall Score.

---

# Stage 2 – Technical Facts

Technical Facts consist of measurable information.

Examples include:

* Length
* Width
* Boot capacity
* Battery capacity
* Charging power
* WLTP range
* Warranty

Technical Facts are never scored directly.

They establish the factual foundation for later evaluation.

---

# Stage 3 – Evidence

Evidence represents documented observations supported by one or more Sources.

Examples include:

* Measured cabin noise
* Verified charging curve
* Independent winter range
* OTA update history

Every Evidence record shall include:

* supporting Source;
* confidence level;
* documented observation.

Evidence documents what is known.

Evidence never contains interpretation.

---

# Stage 4 – Review

Reviews transform Evidence into qualitative understanding.

Examples include:

* Ride comfort
* Cabin refinement
* Software quality
* Steering precision
* Infotainment usability

Every Review shall include:

* interpretation;
* confidence;
* supporting Evidence.

Reviews explain what the Evidence means.

Reviews never replace Evidence.

---

# Stage 5 – Criterion Score

Each Criterion evaluates one aspect of ownership.

Examples include:

* Driving Experience
* Technology
* Long-Term Ownership
* Practicality
* Price / Value

Each Criterion Score references:

* one Configuration;
* one Criterion;
* one Review;
* one Framework Version.

Criterion Scores remain independently explainable.

---

# Stage 6 – Overall Score

The Overall Score combines all weighted Criterion Scores.

The Overall Score introduces no new information.

It aggregates existing Criterion Scores according to the active Framework Version.

The Overall Score is therefore fully reproducible.

---

# Informational Criteria

Some characteristics are documented without materially influencing the Overall Score.

Examples include:

* Ambient lighting
* Premium audio
* Panoramic roof
* Decorative interior features

These characteristics help distinguish otherwise similar Configurations but should not dominate the recommendation.

---

# Excluded Criteria

The following characteristics shall not contribute to framework scoring.

* 0–100 km/h acceleration
* Top speed
* Number of displays
* Marketing claims
* Influencer opinions
* Launch hype

These characteristics may be documented but shall never influence the Overall Score.

---

# Confidence Model

Every Review shall include a confidence level.

| Level   | Meaning                                             |
| ------- | --------------------------------------------------- |
| High    | Multiple independent sources support the conclusion |
| Medium  | Some supporting Evidence is available               |
| Low     | Limited supporting Evidence                         |
| Unknown | Insufficient Evidence is available                  |

Unknown shall always be preferred over unsupported assumptions.

Confidence reflects the quality of the supporting Evidence rather than the strength of the conclusion.

---

# Explainability

Every Overall Score shall remain fully explainable.

The framework shall always be able to answer:

* Which Criteria contributed positively?
* Which Criteria reduced the score?
* Which Reviews supported each Criterion?
* Which Evidence supported each Review?
* Which Sources supported the Evidence?

No score shall exist without a complete explanation path.

---

# Weight Management

Criteria and weighting belong to the Framework rather than individual Configurations.

Changing weighting creates a new Framework Version.

Configuration evaluations generated using different Framework Versions shall never be compared directly.

Historical evaluations remain immutable.

---

# Decision Logic

The framework follows the same decision process for every Configuration.

1. Hard Requirements determine eligibility.
2. Technical Facts establish the factual baseline.
3. Evidence documents observations.
4. Reviews interpret Evidence.
5. Criterion Scores evaluate Reviews.
6. The Overall Score aggregates weighted Criterion Scores.
7. Personal preference may determine the final decision only when competing Configurations receive comparable Overall Scores.

---

# Framework Integrity

To preserve consistency:

* Technical Facts shall never be rewritten without justification.
* Evidence shall never be removed without traceability.
* Reviews shall remain linked to supporting Evidence.
* Every significant methodology change creates a new Framework Version.
* Every Overall Score remains permanently associated with the Framework Version under which it was generated.

---

# Determinism

Given:

* identical Framework Version;
* identical Technical Facts;
* identical Evidence;
* identical Reviews;

the framework shall always produce identical Criterion Scores and Overall Scores.

Deterministic behaviour is a mandatory property of the framework.

---

# Guiding Principle

> **The scoring model does not evaluate vehicles directly. It evaluates Configurations by transforming documented facts into evidence, evidence into understanding, understanding into criterion scores and criterion scores into explainable recommendations.**
