# ADR-008 — Review Ownership Clarification

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.5 (unchanged)

**Supersedes:** None

**Related:**

* ADR-005 — Scoring Mechanics
* audits/ARCH-008-Review-Scoring-Ownership-Evaluation.md
* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md
* docs/03_scoring-model.md

---

# Context

ARCH-008 evaluated whether `Review` owns only textual assessment, conclusion, and confidence, or has begun absorbing concepts that belong to `Scoring` — weight, normalization, total score. It found the schema itself is already correctly separated: `Review.Score` (1–5) is Review's conclusion, and `10_Scoring.RawScore`/`WeightedScore`/`12_OverallScores.OverallScore` are formulas derived from it, never duplicated back onto Review.

Two gaps survived that finding, both small and non-structural:

1. `framework/architecture/entity-model.md`'s `Review` entity lists its `Owns` as `category, summary, confidence` — omitting `score` — even though ADR-005 and `workbook-schema.md` both already document `Score` as a Review-owned column. Read in isolation, `entity-model.md` would lead a contributor to conclude Review has no numeric conclusion at all, which is false.
2. Two of the 100 live `07_Reviews.Summary` values reason about concepts that are not Review's job to narrate: `REV_000007` states a criterion is "lavt vektet i denne kriteriemodellen" (low-weighted in this criterion model), restating `01_Criteria.Weight` inside a Review; `REV_000041` states a Configuration "ikke skulle vaert vektet evaluert i utgangspunktet" (should not have been weighted-evaluated at all), restating a Hard-Requirement/pipeline judgment already recorded in `11_DecisionLog` (`DEC_M002_VIOLATION`) and `12_OverallScores.Notes`.

Neither gap required a schema, column, or worksheet change — the correct fix is documentation accuracy and two rewordings, not architecture.

---

# Decision

`entity-model.md`'s `Review` entity `Owns` list is corrected to `category, score, summary, confidence`, matching what ADR-005 and `workbook-schema.md` already establish.

`workbook-schema.md`'s `07_Reviews` Notes gain a documented convention: **Review Summaries shall not restate Criterion Weight or Hard-Requirement/pipeline-eligibility status.** Both already have a queryable home (`01_Criteria.Weight`; `11_DecisionLog` and the affected Configuration's or Overall Score's Notes) and shall be referenced by pointer, not repeated in prose.

`REV_000007` and `REV_000041`'s `Summary` values are reworded to remove weight/pipeline-eligibility reasoning while preserving their evidence-based conclusions unchanged (`Score`, `Confidence`, and `EvidenceID` links are untouched):

* `REV_000007`: drops "men ladehastigheten er uansett lavt vektet i denne kriteriemodellen"; keeps the DC-charging-spec and consumption interpretation relative to the estimated annual driving distance.
* `REV_000041`: drops "og et symptom på at denne konfigurasjonen ikke skulle vært vektet evaluert i utgangspunktet"; keeps the 220mm-over-limit finding and the resulting lowest-possible score, replacing the dropped clause with a plain pointer to `DEC_M002_VIOLATION` for how the violation is handled in the evaluation, rather than Review asserting the handling itself.

---

# Rationale

`docs/03_scoring-model.md` already states the boundary this ADR enforces: "Reviews interpret Evidence, they do not simulate false precision," with weight ownership assigned entirely to Criterion Score (Stage 5). The two flagged Reviews did not violate this because the schema allowed it — nothing forces a Review's free-text `Summary` to reference weight or pipeline status — but because nothing documented that it shouldn't. Adding the convention closes that gap without constraining `Summary`'s free-text nature for any other purpose.

Fixing `entity-model.md` is a documentation correction, not a decision about what Review should own — ADR-005 already made that decision; `entity-model.md` simply never caught up to it.

---

# Consequences

## Entity Model

`entity-model.md`'s `Review` entity: `Owns` list corrected to include `score`.

## Workbook Schema

`workbook-schema.md`'s `07_Reviews` Notes gain the Review-Summary convention stated above.

## Reference Workbook

`07_Reviews.Summary` for `REV_000007` and `REV_000041` reworded as described. `Score`, `Confidence`, `Category`, `EvidenceID`, and `FrameworkVersion` are unchanged for both rows — this is a wording correction, not a re-evaluation.

No Framework Version increment. Nothing about the scoring methodology, calculation, or any Score/WeightedScore/OverallScore value changes — consistent with prior in-place Review wording revisions in this workbook (e.g. the Kia W018 revision), which also did not bump FrameworkVersion.

---

# Alternatives Considered

## Add a structural constraint preventing Summary from referencing weight/status concepts

Rejected. `Summary` is documented free text by design (the same category as the Notes fields under review in the broader Notes-strategy topic); two sentences out of 100 do not justify a structural constraint. A documented convention is proportionate; a schema constraint is not.

## Leave the two Reviews as-is and only fix the documentation gap

Rejected. The documentation gap and the wording drift are the same underlying confusion — "what does Review own" — surfacing in two different places. Fixing only the doc would leave the two Reviews as a live, real-data counterexample to the convention this ADR establishes.

---

# Impact

Affected documents:

* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md

Affected workbook:

* `07_Reviews` (Summary wording for REV_000007 and REV_000041 only)

No changes are required to:

* implementation-contract.md
* relationships.md
* enumerations.md
* data-flow.md
* docs/03_scoring-model.md (already states the boundary correctly)

---

# Guiding Principle

> **Review answers what the evidence means, once. If the same fact needs to be said twice — once by the entity that owns it, once by Review restating it — Review has stopped answering its own question and started answering someone else's.**
