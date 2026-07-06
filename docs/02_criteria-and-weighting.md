# EV Decision Framework

## Criteria & Weighting

**Version:** 1.1
**Status:** Locked
**Last Updated:** 2026-07-06

---

# Purpose

This document defines the evaluation criteria used by the EV Decision Framework.

The framework evaluates **Configurations**, not Vehicle models.

Criteria define **what** is evaluated.

Weighting defines **how much** each criterion contributes to the final recommendation.

The objective is to minimise long-term ownership regret rather than maximise specifications.

The framework is designed around an ownership horizon of approximately **8–10 years**.

All changes to criteria or weighting shall be version controlled.

---

# Design Philosophy

The criteria model follows six principles.

1. Facts before opinions.
2. Long-term ownership over first impressions.
3. Objective information separated from subjective interpretation.
4. Every score shall be explainable.
5. Unknown is preferable to assumed.
6. Personal preference belongs at the end of the decision process.

---

# Evaluation Model

The framework intentionally separates different stages of evaluation.

## Stage 1 – Facts

Objectively measurable information.

Examples

* Vehicle dimensions
* Battery capacity
* Charging performance
* WLTP range
* Purchase price

Facts never contain interpretation.

---

## Stage 2 – Evidence

Verified observations supported by one or more Sources.

Examples

* Measured cabin noise
* Independent winter range
* Verified charging curve
* OTA update history

Evidence documents what is known.

Evidence never assigns scores.

---

## Stage 3 – Review

Qualitative interpretation of documented Evidence.

Examples

* Driving refinement
* Software quality
* Cabin comfort
* User interface quality

Reviews interpret Evidence.

Reviews never replace Evidence.

---

## Stage 4 – Criteria

Framework criteria evaluate Reviews.

Criteria determine what contributes to the final recommendation.

---

## Stage 5 – Weighting

Weighting expresses the relative importance of each Criterion.

Weighting reflects the framework philosophy rather than objective truth.

---

# Hard Requirements

The following requirements are mandatory.

| Requirement       | Description                                     |
| ----------------- | ----------------------------------------------- |
| Vehicle Type      | Battery electric vehicle (BEV)                  |
| Maximum Length    | 4500 mm                                         |
| Ownership Horizon | Approximately 8–10 years                        |
| Primary Vehicle   | Yes                                             |
| Winter Range      | Approximately 300 km motorway driving in winter |
| Market            | Norway                                          |

Configurations failing a mandatory requirement shall not proceed to weighted evaluation.

---

# Weighted Categories

The following categories contribute to the Overall Score.

---

## 1. Driving Experience

Importance

**Very High**

Examples

* Cabin noise
* Ride comfort
* Steering precision
* Visibility
* Driving ergonomics

Focus

Long-distance comfort, confidence and everyday usability.

---

## 2. Technology

Importance

**Very High**

Examples

* OTA updates
* Apple CarPlay
* Software responsiveness
* Navigation
* User interface
* 360 Camera

Focus

Technology should improve ownership rather than impress during a short demonstration.

---

## 3. Perceived Quality

Importance

**High**

Examples

* Materials
* Build quality
* Cabin refinement
* Controls
* Software polish

Focus

The Configuration should feel well engineered throughout the ownership period.

---

## 4. Long-Term Ownership

Importance

**High**

Examples

* Reliability
* Software support
* Warranty
* Brand maturity
* Nordic suitability

Focus

Long-term ownership quality is more important than launch-day excitement.

---

## 5. Practicality

Importance

**Medium**

Examples

* Cargo capacity
* Rear-seat usability
* Everyday practicality

Focus

Daily convenience should outweigh occasional edge cases.

---

## 6. Price / Value

Importance

**Medium**

Price alone is never the objective.

The framework evaluates long-term value rather than lowest purchase price.

A more expensive Configuration may score higher if it meaningfully improves the ownership experience.

---

## 7. Charging & Efficiency

Importance

**Medium**

Examples

* Winter efficiency
* Charging speed
* Route planning

Focus

Charging performance should support predictable long-distance travel under Norwegian conditions.

---

## 8. Garage Friendliness Beyond Minimum

Importance

**Medium**

Garage friendliness is evaluated in two stages. Whether a Configuration fits the owner's garage at all is a Hard Requirement (maximum vehicle length, see Hard Requirements). This category evaluates the practical margin a Configuration provides beyond that minimum.

Examples

* Length margin below the maximum requirement
* Turning and manoeuvring space within the garage

Focus

A Configuration that clears the hard length requirement with room to spare reduces day-to-day friction over the ownership period, independent of general cargo or rear-seat practicality.

---

# Informational Criteria

Some features are recorded because they contribute to the ownership experience, but they have little or no influence on the Overall Score.

Examples include:

* Ambient lighting
* Panoramic roof
* Premium audio
* Ventilated seats
* Electric tailgate

These features may help distinguish otherwise similar Configurations.

---

# Excluded Criteria

The following shall not contribute to framework scoring.

* 0–100 km/h acceleration
* Top speed
* Number of displays
* Marketing hype
* Social media popularity
* First impression "wow factor"

These characteristics may be interesting but are not considered meaningful predictors of long-term ownership satisfaction.

---

# Decision Rule

The framework provides decision support.

The purchaser makes the final decision.

If two Configurations receive comparable Overall Scores, subjective preference may determine the final choice.

If one Configuration receives a clearly stronger evaluation, the framework should encourage reconsideration of personal bias before overriding the result.

---

# Evidence Policy

Every Review shall be supported by documented Evidence.

Preferred Evidence hierarchy:

## Tier A

Objective and independently verifiable sources.

Examples:

* Manufacturer documentation
* Euro NCAP
* EV Database
* ADAC
* Independent winter testing

---

## Tier B

Professional automotive reviews.

---

## Tier C

Long-term ownership experience.

No individual source shall determine an evaluation on its own.

Confidence should increase when multiple independent sources support the same conclusion.

---

# Confidence

Every qualitative Review shall include a confidence level.

| Level   | Meaning                                             |
| ------- | --------------------------------------------------- |
| High    | Multiple independent sources support the conclusion |
| Medium  | Some supporting Evidence is available               |
| Low     | Limited supporting Evidence                         |
| Unknown | Insufficient Evidence is available                  |

Unknown shall always be preferred over unsupported assumptions.

---

# Versioning

Criteria and weighting shall be locked before evaluating Configurations.

Changes require a new Framework Version.

Historical Configuration evaluations shall remain associated with the Framework Version under which they were produced.

Scores generated using different Framework Versions shall never be compared directly.

---

# Guiding Principle

> **Criteria define what matters. Weighting expresses priorities. Evidence supports conclusions. The framework evaluates Configurations while preserving transparency, explainability and long-term ownership focus.**
