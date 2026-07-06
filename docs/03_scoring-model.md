# EV Decision Framework

## Scoring Model

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines how vehicle evaluations are calculated.

The scoring model is designed to produce transparent, explainable and repeatable recommendations.

Every score should be reproducible using the same input data and framework version.

---

# Evaluation Pipeline

Vehicle evaluation follows the same sequence for every candidate.

```
Vehicle

↓

Hard Requirements

↓

Objective Data

↓

Evidence Collection

↓

Quality Assessment

↓

Weighted Scoring

↓

Final Recommendation
```

Each stage is independent from the next.

---

# Stage 1 – Hard Requirements

Hard requirements act as filters.

Vehicles failing one or more mandatory requirements are excluded before scoring begins.

Examples include:

* Maximum vehicle length
* Vehicle type
* Winter range requirement
* Market availability

A vehicle that fails a hard requirement does not receive a final score.

---

# Stage 2 – Objective Data

Objective data consists of measurable facts.

Examples:

* Length
* Width
* Boot capacity
* Battery size
* Charging speed
* WLTP range
* Warranty

Objective data is never manually scored.

It serves as the factual foundation for later analysis.

---

# Stage 3 – Evidence Collection

Evidence supports qualitative assessments.

Every claim should be traceable to one or more sources.

Examples:

Claim:

> Cabin is quiet at motorway speed.

Supporting evidence:

* Professional measurements
* Multiple independent reviews
* Owner reports

Evidence should always include:

* Source
* Date
* Confidence

---

# Stage 4 – Quality Assessment

Some characteristics cannot be measured directly.

Examples:

* Software quality
* Cabin refinement
* Adaptive cruise behaviour
* Steering feel
* Infotainment usability

These characteristics receive evaluation scores.

Every qualitative score must include:

* Score
* Confidence
* Supporting evidence

---

# Stage 5 – Weighted Scoring

Only weighted criteria contribute to the total score.

Each criterion receives:

* Weight
* Raw score
* Weighted score

Weights are maintained separately from the vehicle data.

Changing weights never changes historical measurements.

---

# Informational Criteria

Some attributes are tracked without significantly affecting ranking.

Examples:

* Ambient lighting
* Premium audio
* Panoramic roof
* Decorative interior features

These improve documentation but should have little or no influence on the final recommendation.

---

# Excluded Criteria

The following attributes are intentionally excluded from scoring:

* 0–100 km/h
* Top speed
* Number of displays
* Marketing claims
* Influencer opinions
* Launch hype

These may be documented but never influence ranking.

---

# Confidence Model

Every qualitative assessment receives a confidence level.

## High

Supported by multiple independent high-quality sources.

## Medium

Supported by limited evidence.

## Low

Preliminary information only.

## Unknown

No reliable evidence currently available.

Unknown values are preferred over assumptions.

---

# Explainability

Every final score must be explainable.

The framework should always be able to answer:

* Which criteria contributed positively?
* Which criteria reduced the score?
* Which evidence supports each conclusion?

No score should exist without explanation.

---

# Weight Management

Weights belong to the framework rather than individual vehicles.

Changing weights creates a new framework version.

Vehicle evaluations should never mix weighting models.

---

# Decision Logic

The framework follows these rules:

1. Hard requirements determine eligibility.
2. Objective facts establish the baseline.
3. Evidence supports qualitative assessment.
4. Weighted criteria produce the final score.
5. Subjective preference may decide only when competing vehicles receive similar overall scores.

---

# Framework Integrity

To preserve consistency:

* Historical data should never be rewritten.
* Evidence should never be discarded without reason.
* Every significant methodology change creates a new framework version.
* Vehicle evaluations remain tied to the framework version under which they were produced.

---

# Design Objective

The scoring model exists to support better decisions through transparency rather than mathematical complexity.

The preferred vehicle is not necessarily the one with the highest specification sheet.

It is the vehicle that best satisfies the defined requirements, supported by objective evidence and evaluated through a consistent methodology.
