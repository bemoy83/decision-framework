# ADR-010 — Framework Versioning Rule

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.7 (Next)

**Supersedes:** None

**Related:**

* ADR-004, ADR-005, ADR-006, ADR-007, ADR-008, ADR-009
* audits/ARCH-010-Framework-Versioning-Lifecycle-Evaluation.md
* docs/02_criteria-and-weighting.md
* docs/03_scoring-model.md
* framework/architecture/entity-model.md
* framework/architecture/enumerations.md
* framework/architecture/workbook-schema.md

---

# Context

ARCH-010 asked what happens when `FrameworkVersion` increments, whether existing Configurations must be re-scored, and whether the same dataset can legitimately exist under more than one Framework Version at once. It found the docs gesture at an answer ("every significant methodology change creates a new Framework Version") without defining "significant" or "methodology," and reconstructed the framework's actual history to find the real, already-consistent pattern underneath: every one of the seven historical version bumps (0.2→1.0→1.2→1.3→1.4→1.5→1.6) was a **schema** change (new/changed worksheet, column, or enumeration) except ADR-005, which bootstrapped the scoring calculation engine itself. Not one of the framework's 29 Criteria has ever had its `Weight`, `Type`, or `HardRequirement` flag changed since inception. The live workbook also already demonstrates mixed-version coexistence working correctly: 7 `07_Reviews` rows remain stamped `FrameworkVersion 1.2` while their dependent `10_Scoring` rows are stamped `1.4`, because ADR-005 later formalized how Score is calculated without needing to rewrite the Review's own qualitative content.

The audit's conclusion, confirming the user's own hypothesis: no change to the versioning model is needed. Only the rule needs to be written down.

---

# Decision

**A `FrameworkVersion` increment is triggered by exactly one of two kinds of change:**

1. **Schema change** — a new/changed/removed worksheet or column, a new enumeration, or a documentation correction that does not alter any existing Criterion's `Weight`, `Type`, `HardRequirement` flag, or the `RawScore`/`WeightedScore` calculation rule. No existing Review, Score, or OverallScore becomes invalid; all remain fully comparable to evaluations produced after the bump.
2. **Methodology change** — a change to a Criterion's `Weight`, `Type`, `HardRequirement` flag, or Active status, to the `RawScore`/`WeightedScore` calculation rule, or to a Hard Requirement's own pass/fail threshold. Only Scores for the *affected* Criteria become non-comparable to Scores for those same Criteria produced after the bump; unaffected Criteria and Configurations are unaffected.

To determine whether an existing evaluation needs re-scoring after a bump: check for a Methodology-type entry in `11_DecisionLog` between the evaluation's own `FrameworkVersion` (exclusive) and the workbook's current `FrameworkVersion` (inclusive). None exists today between any Configuration's own recorded version and `1.6`, so no existing Configuration requires re-scoring as of this ADR.

A row's `FrameworkVersion` records the ruleset that produced *that row's own value* — not necessarily when its underlying facts were first gathered. A Review and its dependent Score may legitimately carry different `FrameworkVersion` values, as already demonstrated by the live `07_Reviews`/`10_Scoring` data.

`11_DecisionLog` gains an optional `VersionBumpType` column: `SCHEMA`, `METHODOLOGY`, or `NONE` (for entries that did not change the active version at all). This turns the "scan DecisionLog" step above into a filter rather than a prose read. All 19 existing `11_DecisionLog` entries are backfilled per the ledger in ARCH-010.

---

# Rationale

The rule is not new policy — it is a description of behavior the framework has followed consistently since its first version bump, made explicit so it does not have to be independently re-derived (as this audit had to do) the next time the question comes up. Stating it as "schema vs. methodology" rather than "significant vs. insignificant" replaces a subjective judgment call with a mechanical check: does this change touch a Criterion's `Weight`/`Type`/`HardRequirement` flag or the scoring formula, yes or no.

`VersionBumpType` is optional relative to the rule itself — the rule is fully usable by reading `11_DecisionLog`'s existing `Change` text, exactly as ARCH-010 did. It is adopted anyway because the cost is low (one column, backfilled once) and it converts a manual audit step into something any future contributor or tool can filter mechanically.

---

# Consequences

## Documentation

* `docs/02_criteria-and-weighting.md`'s Versioning section replaces its single undefined line with the two-kind rule and its consequence.
* `docs/03_scoring-model.md`'s Framework Integrity section gains the same rule, plus the DecisionLog-scan procedure for determining whether re-scoring is needed.
* `framework/architecture/entity-model.md`'s `FrameworkVersion` entity gains a note that a row's `FrameworkVersion` reflects the ruleset that produced it, not necessarily when its facts were gathered, using the live Review-1.2/Score-1.4 coexistence as the worked example.
* `framework/architecture/enumerations.md` gains a **Version Bump Type** enumeration (`SCHEMA`/`METHODOLOGY`/`NONE`).
* `framework/architecture/workbook-schema.md`'s `11_DecisionLog` section gains the `VersionBumpType` column.

## Reference Workbook

* `11_DecisionLog` gains `VersionBumpType`, backfilled for all 19 existing rows per the ARCH-010 ledger.
* README gains `Enum_VersionBumpType`; `WorkbookVersion` 1.5 → 1.6; `FrameworkVersion` 1.6 → 1.7 (this ADR is itself a Schema-type bump under the rule it documents — no existing Configuration requires re-scoring as a result, which this ADR's own rule confirms).

---

# Alternatives Considered

## Leave the rule as informal prose, not a documented two-kind distinction

Rejected. This is the status quo ARCH-010 evaluated and found insufficient — "significant methodology change" has no operational definition today, which is exactly what prompted this review.

## Make `VersionBumpType` a mandatory column with strict validation blocking un-typed entries

Rejected as disproportionate. The column is a filtering convenience, not a integrity-critical reference; making it mandatory would require retrofitting a firm judgment onto older entries with more ceremony than the column's own value justifies.

## Split `FrameworkVersion` into two separate version numbers (schema version and methodology version)

Rejected, consistent with the user's own hypothesis and ARCH-010's conclusion — this would be a real model change where none is needed. `WorkbookVersion` already tracks a closely related but distinct concept (the reference workbook's own structural version); introducing a third number was judged to add more confusion than clarity for a distinction that a `VersionBumpType` column on the existing `FrameworkVersion` timeline already resolves.

---

# Migration Strategy

No historical data changes meaning. All 19 `11_DecisionLog` rows are backfilled with `VersionBumpType` reflecting the classification already reconstructed in ARCH-010; no `Change`, `Reason`, `DecisionOwner`, or `Notes` text is altered.

---

# Impact

Affected documents:

* docs/02_criteria-and-weighting.md
* docs/03_scoring-model.md
* framework/architecture/entity-model.md
* framework/architecture/enumerations.md
* framework/architecture/workbook-schema.md

Affected workbook:

* `11_DecisionLog` (`VersionBumpType` column added and backfilled)
* README (`Enum_VersionBumpType`, `WorkbookVersion`, `FrameworkVersion`)

No changes are required to:

* `01_Criteria`, `10_Scoring`, `12_OverallScores` — no Criterion weight or calculation rule changes; per this ADR's own rule, no re-scoring is triggered.

---

# Guiding Principle

> **A version number that climbs for reasons no one wrote down eventually stops meaning anything. Every increment should answer, without debate, whether what came before it is still comparable to what comes after.**
