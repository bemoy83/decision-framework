# ARCH-008 — Review & Scoring Ownership Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.5
**Related:** ADR-003, ADR-005, entity-model.md, workbook-schema.md, relationships.md, docs/03_scoring-model.md, ARCH-006, ARCH-007

---

# Question Being Answered

What does `Review` actually own? Specifically: does it own textual assessment, conclusion, and confidence — or has it started absorbing weight, normalization, or total-score concepts that belong to `Scoring`?

---

# Executive Summary

At the **schema level**, the boundary is clean and already matches the ideal stated for this review: `Review` owns `Category`, `Score` (a 1–5 qualitative-to-numeric conclusion), `Confidence`, and `Summary`; it does not store, reference, or duplicate `Criterion.Weight`, `RawScore`, `WeightedScore`, or `OverallScore` anywhere in its structured columns. `10_Scoring.RawScore` is a live formula that reads `Review.Score` and normalizes it (`ROUND(Score/5*100, 2)`); `WeightedScore` multiplies that by `Criterion.Weight` looked up from `01_Criteria`. Nothing about weight or normalization is stored on `Review` — it is computed downstream, exactly once, per ADR-005. `docs/03_scoring-model.md` states this boundary explicitly: "Reviews interpret Evidence, they do not simulate false precision," with weight ownership assigned entirely to Stage 5 (Criterion Score).

One real, pre-existing documentation gap was found: `entity-model.md`'s `Review` entity lists its `Owns` as `category, summary, confidence` — omitting `Score` entirely — while `workbook-schema.md` and ADR-005 both correctly document `Score` as a Review-owned column. This is a doc-internal contradiction about whether Review owns a score at all, not evidence of scoring creeping into Review; it should be corrected by adding `score` to Review's documented ownership, reconciling `entity-model.md` with the ADR-005 decision that already governs.

The concrete "creep" the user sensed is real, but it lives in **free text, not structure**: two of the 100 existing `07_Reviews.Summary` values reason explicitly about scoring/methodology concepts that are not Review's job to narrate:

* `REV_000007` (W020, Rekkevidde/lading) states charging speed is "lavt vektet i denne kriteriemodellen" (low-weighted in this criterion model) — restating `01_Criteria.Weight` (which is in fact 2/100 for W020, so the claim is accurate) inside Review's interpretation of the evidence, rather than leaving weight exclusively to Criterion.
* `REV_000041` (W019, Garasje) states this Configuration "ikke skulle vaert vektet evaluert i utgangspunktet" (should not have been weighted-evaluated at all) — a Hard-Requirement/pipeline-eligibility judgment (M002 violation), correctly already recorded in `11_DecisionLog` (`DEC_M002_VIOLATION`) and `12_OverallScores.Notes`, but restated here as if it were also Review's call to make.

No other Review row in the live data (100 checked) shows this pattern. `Review.Category` was also checked for drift against its linked `Criterion.Category` across all 100 Score-linked Reviews — zero mismatches — so the theoretical Category-drift risk flagged before this review began did not materialize.

---

# Current Assessment

**Structurally Sound, Two Free-Text Boundary Violations.**

The entity/column-level separation between Review and Scoring is exactly what ADR-005 and `docs/03_scoring-model.md` prescribe, and the live data confirms it holds: no Review row stores a weight, a normalized value, or an aggregate. `Score.Explanation` (10_Scoring) does restate the RawScore/WeightedScore arithmetic in prose on every one of its 100 rows (e.g. "WeightedScore = RawScore x Weight(8)/100 = 6.4") — this is Score narrating its own already-self-documenting formula, not Review absorbing Scoring's job, so it is out of scope for this specific ownership question, but is a related, lower-priority duplication worth folding into the P5 Notes-strategy review (the prose can silently go stale if a Criterion's Weight is ever revised, since the formula recalculates but the sentence does not).

It is not rated "Fully Sound" because of the two Review Summary rows above. Both are narrow, specific, and easy to reword — they are not evidence the schema needs to change, only that nothing currently stops a contributor from writing weight- or pipeline-aware reasoning into a field whose documented job is "what does the evidence mean."

---

# Ownership Boundary Review

| Concept | Where it lives today | Verdict |
| --- | --- | --- |
| Textual assessment ("what does the evidence mean") | `Review.Summary` | **Correctly owned by Review.** 98 of 100 Summary rows stay within this boundary. |
| Conclusion (qualitative judgment, expressed numerically) | `Review.Score` (1–5) | **Correctly owned by Review**, per ADR-005 and `docs/03_scoring-model.md` Stage 4. `entity-model.md`'s omission of `score` from Review's `Owns` list is a documentation gap, not a structural one — the ADR and the live workbook both already treat it as Review's. |
| Confidence | `Review.Confidence` | **Correctly owned by Review.** No drift found. |
| Category | `Review.Category` | **Correctly owned by Review**, verified against `Criterion.Category` for all 100 linked rows — zero mismatches. |
| Criterion weight | `01_Criteria.Weight` | **Correctly owned by Criterion**, exclusively — except two Review Summary rows narrate a criterion's weight in prose (REV_000007, REV_000041). |
| Normalization (Score → 0–100 scale) | `10_Scoring.RawScore` (formula) | **Correctly owned by Scoring.** No normalized value exists anywhere on Review. |
| Weighted score | `10_Scoring.WeightedScore` (formula) | **Correctly owned by Scoring.** Not duplicated on Review. |
| Total / Overall score | `12_OverallScores.OverallScore` (formula) | **Correctly owned by OverallScore**, per ADR-005. No Review or Score row references it. |
| Hard Requirement / pipeline-eligibility status | `11_DecisionLog` + `12_OverallScores.Notes` (prose, per P3) | **Correctly recorded elsewhere already** — but REV_000041 restates the same judgment inside a Review Summary, which is not Review's documented job and duplicates a decision that already has a home. |

---

# Risks

**Risk of treating this as a schema problem and changing Review's structure:**

* There is no structural violation to fix. Adding constraints (e.g. forbidding certain words in `Summary`) at the schema level would be disproportionate to two free-text sentences, and `workbook-schema.md`'s Notes fields are documented as free text by design (see the P5 Notes-strategy topic already queued in the broader review) — Review.Summary is conceptually the same kind of field.

**Risk of leaving the free-text drift uncorrected:**

* The two existing sentences will keep reading as architecturally confusing to any future contributor trying to answer "what does Review own?" from the live data rather than the docs — exactly the question that prompted this review.
* Without a documented convention (e.g. "Review.Summary shall not reference Criterion.Weight or Hard-Requirement/pipeline status — cite the Criterion or DecisionLog entry instead"), nothing stops a third instance from appearing as more Reviews are written for future vehicles.
* Leaving `entity-model.md`'s Review `Owns` list out of sync with ADR-005 and `workbook-schema.md` means a future contributor reading only `entity-model.md` (the entity-level summary) would incorrectly conclude Review has no numeric conclusion at all.

---

# Recommendation

**No schema or methodology change required. Two small, low-risk corrections recommended, both documentation/data-quality, not architecture:**

1. Add `score` to `Review`'s documented `Owns` list in `entity-model.md`, reconciling it with ADR-005 and `workbook-schema.md`, which already treat `Review.Score` as Review's 1–5 conclusion.
2. Reword `REV_000007` and `REV_000041`'s Summary text to remove the weight/pipeline-eligibility reasoning, keeping each Review confined to interpreting its own evidence (charging speed adequacy; the 4720mm length fact) and pointing to `01_Criteria.Weight` / `11_DecisionLog` by reference rather than restating their content in prose.
3. Optionally, add a one-line convention to `workbook-schema.md`'s `07_Reviews` Notes: Review Summaries shall not restate Criterion Weight or Hard-Requirement/pipeline status — both already have a queryable home elsewhere.

This does not require a Framework Version increment — nothing about the calculation methodology changes, only documentation accuracy and two Reviews' wording (consistent with how prior in-place Review wording revisions in this workbook, e.g. the Kia W018 revision, did not bump FrameworkVersion either).

---

# Definition of Done

* [x] `Review` and `Score`/`Scoring` evaluated against `entity-model.md`, `workbook-schema.md`, `relationships.md`, `docs/03_scoring-model.md`, and ADR-005.
* [x] All 100 live `07_Reviews` rows checked for scoring/weight/pipeline language in `Summary`.
* [x] All 100 Score-linked Reviews checked for `Category` drift against their linked `Criterion.Category` — zero mismatches found.
* [x] `10_Scoring` formulas inspected directly to confirm `RawScore`/`WeightedScore` derive from `Review.Score` and `Criterion.Weight` with no duplicated or independently-entered weight/normalization value anywhere on Review.
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `entity-model.md`, `workbook-schema.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: fix the `entity-model.md` documentation gap and reword two specific Review Summaries; no schema change, no Framework Version bump.
