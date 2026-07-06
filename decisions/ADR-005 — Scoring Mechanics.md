# ADR-005 — Scoring Mechanics: Numeric Scale, Calculation Engine, and Overall Score Aggregation

**Status:** Accepted

**Date:** 2026-07-06

**Framework Version:** 1.4 (Next)

**Supersedes:** None

**Related:**

* ADR-002 — Reference Implementation
* ADR-003 — Configuration Is The Evaluated Entity
* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md
* framework/architecture/implementation-contract.md
* docs/03_scoring-model.md

---

# Context

A "thin vertical slice" exercise took one Configuration (`TESLA_MODEL_3_LONG_RANGE_RWD`) through the full Evidence → Review → Criterion Score pipeline with real, sourced Reviews, to validate that the documented scoring methodology actually produces a coherent, explainable result.

It technically worked — every Score record validated with zero referential-integrity or enumeration errors — but running it exposed three gaps that only become visible once real numbers are pushed through the pipeline.

**No documented numeric scale.** `docs/03_scoring-model.md` and `framework/architecture/workbook-schema.md` describe `Review.Score`, `10_Scoring.RawScore` and `10_Scoring.WeightedScore` as concepts, but neither document states what scale `Review.Score` is recorded on, nor how `RawScore` is normalized from it. During the vertical slice, a 1–5 integer scale for `Review.Score` and a `RawScore = Score / 5 × 100` normalization were used ad hoc and documented only inline in each Score record's free-text `Explanation` field.

**No calculation engine.** `10_Scoring` has column headers only; no formulas, no script, anywhere in the repository implement the calculation. Every `RawScore`/`WeightedScore` value produced during the vertical slice was computed by a one-off script and pasted into the sheet as a static value. `implementation-contract.md`'s Determinism section requires that "given identical FrameworkVersion, identical data, identical Evidence, identical Reviews, every compliant implementation shall produce identical results" — a requirement that cannot be verified, let alone guaranteed, while the only "engine" is a script run once by hand.

**No Overall Score aggregation exists.** `implementation-contract.md`'s Scoring section requires implementations to calculate "Criterion Scores; Overall Scores." `docs/03_scoring-model.md` Stage 6 and `framework/architecture/data-flow.md` Stage 8 both describe Overall Score as a real pipeline stage that "aggregates existing Criterion Scores... introduces no new information." Despite this, no entity, worksheet, or column anywhere stores or computes it — only per-criterion Scores are modeled (`entity-model.md`'s `Score` entity references exactly one `Review`, which is definitionally a per-criterion relationship). During the vertical slice, only 7 of 20 `WEIGHTED` criteria (37 of 100 weight-points) were scored for the one Configuration tested. No Overall Score was written, specifically to avoid presenting a partial sum as a complete evaluation — a workaround, not a mechanism.

---

# Decision

**Numeric scale.** `Review.Score` is a fixed integer scale from 1 to 5 (ratifying the convention already used). `RawScore = ROUND(Score / 5 × 100, 2)`. `WeightedScore = ROUND(RawScore × Criterion.Weight / 100, 2)`. The rounding rule is explicit and binding so that any compliant calculation of the same inputs produces byte-identical output.

**Calculation engine.** `10_Scoring.RawScore` and `10_Scoring.WeightedScore` shall be Excel formulas — looking up `Review.Score` from `07_Reviews` and `Criterion.Weight` from `01_Criteria` by ID — not manually entered or pasted values. This keeps the Reference Workbook's scores self-recalculating and inspectable, consistent with ADR-002's rationale for choosing Excel, and with `workbook-schema.md`'s existing rule that "Scores should never be manually edited."

**Overall Score.** A new entity, **Overall Score**, is introduced, distinct from the per-criterion `Score` entity. It aggregates Criterion Scores for one Configuration under one Framework Version. Unlike `Score`, it does not reference a single `Review` — it references `Configuration` and `FrameworkVersion` only. It is implemented as a new worksheet, `12_OverallScores` (Calculated), with one row per Configuration + Framework Version:

```text
OverallScoreID, ConfigurationID, FrameworkVersion, OverallScore, CoveragePercent, CriteriaScored, CriteriaTotal, Notes
```

`CoveragePercent` = (sum of `Weight` for `WEIGHTED` + `Active` criteria that have a Score for this Configuration) ÷ (sum of `Weight` for all `WEIGHTED` + `Active` criteria) × 100. `OverallScore` is the sum of `WeightedScore` across that Configuration's `10_Scoring` rows. An `OverallScore` shall never be interpreted, displayed, or exported without its `CoveragePercent` alongside it. A `CoveragePercent` below 100 marks the Overall Score as incomplete — it is a real, valid, reproducible number, but not yet a complete evaluation. This directly operationalizes `docs/01_project-philosophy.md`'s "Unknown is preferable to assumed": an incomplete rollup must never be silently read as a finished recommendation.

---

# Rationale

Determinism and reproducibility, both mandatory per `implementation-contract.md`, are not properties a spreadsheet has by default — they have to be built in as formulas that recompute, not as numbers a person once calculated correctly. Ratifying the scale and rounding rule is what makes "identical inputs produce identical outputs" checkable at all, rather than a matter of trusting whoever ran the last script.

Overall Score is not a new concept invented by this ADR — it is a requirement `implementation-contract.md` already states and this ADR finally implements. The `CoveragePercent` field is the one genuinely new addition: without it, a partially-scored Configuration's Overall Score is indistinguishable from a fully-scored one, which would actively mislead rather than merely omit information.

---

# Consequences

## Entity Model

`framework/architecture/entity-model.md` gains an **Overall Score** entity: owns `overall score`, `coverage percentage`; references `Configuration`, `FrameworkVersion`. Distinct from `Score` because it does not reference a single `Review`.

## Relationships

`framework/architecture/relationships.md` gains: Configuration → Overall Score (1:N across Framework Versions); Criterion Score (`10_Scoring`) → Overall Score is an aggregation, not a reference (Overall Score computes from Scores, it does not point back to a specific one).

## Workbook Schema

`framework/architecture/workbook-schema.md` gains a `12_OverallScores` worksheet specification (Calculated type, same governance as `10_Scoring`: never manually edited, always reproducible from underlying `10_Scoring` data). `10_Scoring`'s Notes are updated to state `RawScore`/`WeightedScore` are formulas, not manually entered values.

## Data Flow

`framework/architecture/data-flow.md` Stage 8 (Overall Score) gains a concrete implementation reference to `12_OverallScores` and the `CoveragePercent` safeguard.

## Scoring Model Documentation

`docs/03_scoring-model.md` documents the 1–5 `Review.Score` scale and the `RawScore`/`WeightedScore` rounding rule that is currently missing.

## Reference Workbook

`10_Scoring.RawScore`/`WeightedScore` for the 7 existing vertical-slice records are converted from static values to formulas. A regression check confirms the formulas reproduce the existing values exactly: 25.4 total weighted points across 37 of 100 possible weight-points (37% coverage) for `TESLA_MODEL_3_LONG_RANGE_RWD`. A new `12_OverallScores` worksheet is added with one row reflecting this partial result.

---

# Alternatives Considered

## 0–10 scale instead of 1–5

Rejected. No material benefit, and it would invalidate the vertical slice's already-recorded Review data for no gain.

## Store Overall Score on `03_Configurations` or `10_Scoring`

Rejected. `03_Configurations` is documented to own purchasing identity, not calculated results; `10_Scoring`'s documented purpose is per-criterion Scores. Overloading either worksheet's stated single responsibility (`workbook-schema.md` Design Principle 2) to also hold a Configuration-level rollup would break the contract of an existing worksheet rather than extend it cleanly.

## External script or database as the calculation engine

Rejected, consistent with ADR-002. The Reference Workbook was chosen specifically for manual inspectability ("Contributors can inspect data directly... Calculations can be inspected manually"); a script that runs outside the workbook reintroduces the exact opacity ADR-002 rejected.

## Show Overall Score without a coverage indicator

Rejected. A partial sum presented without its completeness context is indistinguishable from a finished evaluation and would violate "Unknown is preferable to assumed."

---

# Migration Strategy

Framework Version 1.3 workbook data (Criterion Scores as static pasted values) remains historically valid and reproducible under its original methodology. Framework Version 1.4 requires `RawScore`/`WeightedScore` to be formulas and introduces `12_OverallScores`. The 7 existing Score records from the vertical slice are converted to formulas as part of this ADR's implementation; their computed values must not change.

---

# Impact

Affected documents:

* framework/architecture/entity-model.md
* framework/architecture/relationships.md
* framework/architecture/workbook-schema.md
* framework/architecture/data-flow.md
* docs/03_scoring-model.md

Affected workbook:

* `10_Scoring` (formulas replace static values)
* `12_OverallScores` (new worksheet)

No changes are required to:

* implementation-contract.md (already required this; this ADR fulfils that requirement)
* enumerations.md

---

# Guiding Principle

> **A score that cannot recompute itself is not a score, it is a memory of one. An Overall Score that hides how much of the evaluation it actually covers is not a summary, it is a guess wearing a summary's clothes.**
