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

Where no independent winter highway range test exists for a Configuration, Winter Range may be evaluated against an estimate (`WLTP_RANGE x 0.70`) at `Confidence = LOW` instead of remaining `UNKNOWN` indefinitely (ADR-014).

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

## Tier Mapping

`09_Sources.Type` (ADR-012) maps onto this hierarchy as follows:

| Type | Tier |
| --- | --- |
| MANUFACTURER, CERTIFICATION, DATABASE | Tier A |
| GOVERNMENT | Tier A |
| PROFESSIONAL_REVIEW, VIDEO | Tier B |
| COMMUNITY, OWNER_EXPERIENCE | Tier C |
| OTHER | Assessed case by case; default Tier C unless documented otherwise in the Source's own Notes |

Confidence should increase when multiple independent sources support the same conclusion.

## Sourcing Verification (ADR-013)

An AI-generated web-search summary may be used to *locate* a candidate source. It shall never be used to *confirm* what that source says.

A claim shall not be cited as Evidence, or used to support a Review, until its existence and content have been independently verified by direct retrieval of the cited source's own page.

Where direct retrieval cannot confirm the claim — the page does not contain the asserted figure, or the page is inaccessible — the fact shall be recorded as Unknown, per `docs/01_project-philosophy.md`'s "Unknown is better than assumed" principle, not asserted at a merely reduced confidence.

For example: a web-search summary once asserted a specific winter-range test result for a vehicle under evaluation; direct retrieval of the cited source's own page showed no such result at all, only unrelated summer test data. The claim was correctly left Unknown.

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

A Framework Version increment is one of two kinds (ADR-010):

* **Schema change** — a new/changed/removed worksheet or column, a new enumeration, or a documentation correction that does not alter any Criterion's Weight, Type, or Hard Requirement flag, or the scoring calculation rule. No existing evaluation becomes invalid; historical Scores remain fully comparable to Scores produced after the increment.
* **Methodology change** — a change to a Criterion's Weight, Type, or Hard Requirement flag, or to the scoring calculation rule itself. Only Scores for the affected Criteria become non-comparable to Scores for those same Criteria produced after the increment.

Historical Configuration evaluations shall remain associated with the Framework Version under which they were produced.

Scores generated using different Framework Versions shall never be compared directly for the same Criterion, unless the intervening increments were Schema changes only.

A row's Framework Version records the ruleset that produced that row's own value, not necessarily when its underlying facts were gathered — a Review and its dependent Score may legitimately carry different Framework Versions.

---

# Guiding Principle

> **Criteria define what matters. Weighting expresses priorities. Evidence supports conclusions. The framework evaluates Configurations while preserving transparency, explainability and long-term ownership focus.**
