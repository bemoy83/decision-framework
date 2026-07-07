# ARCH-009 — Hard Requirement Model Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.5
**Related:** docs/02_criteria-and-weighting.md, docs/03_scoring-model.md, ADR-004, ADR-005, entity-model.md, workbook-schema.md, enumerations.md, ARCH-006, ARCH-008

---

# Question Being Answered

Today, Hard Requirement compliance is expressed through Configuration Notes prose and manual cross-checking. Should it become its own model — `Configuration → Hard Requirement Result → PASS/FAIL/UNKNOWN → Reason`, as proposed — to make filtering and reporting mechanical instead of narrative?

---

# Executive Summary

**Yes, with one refinement: the Result needs a Confidence field alongside Reason, and it should be modeled as a contributor-interpreted record (like Review) rather than a pure calculated field (like Scoring).**

The evidence for adopting a real model is stronger than the original P3 flag suggested, because the underlying facts each Hard Requirement needs already exist as structured data elsewhere in the workbook — they are simply never joined into a per-Configuration compliance answer:

* **M002** (length ≤ 4500mm) — the raw fact already lives in `04_Technical` (`TF_LENGTH`).
* **M004** (adaptive cruise control) and **M005** (rearview camera) — the raw facts already live in `05_Equipment` (`EQ_ACC`, `EQ_REAR_CAMERA`).
* **M003** (winter range ≈300km) — the raw fact, where gathered, already lives in `04_Technical` (`TF_WINTER_HIGHWAY_RANGE_ESTIMATE`).
* **M001** (battery electric vehicle) — no structured field exists, but every vehicle in this workbook was curated to be a BEV; it has never been at real risk of failing and is not evidence of a gap.

Mechanically joining these existing facts against their thresholds (script-computed for this audit, not stored anywhere in the workbook) produces a full 18-Configuration × 5-Requirement compliance matrix in seconds — the exact filtering/reporting benefit the proposal describes. Today that matrix does not exist anywhere; reconstructing it requires reading 18 separate Notes paragraphs, several of which do not mention a given requirement at all.

The live data also shows exactly why a *pure formula* result (mirroring how `RawScore`/`WeightedScore` are formulas) would be the wrong shape: a naive `Winter range ≥ 300` comparison on Renault 4 Techno's only available data point (287km) computes a confident **FAIL**, but the actual evidence is a mismatched-trim, mild-weather proxy measurement — the framework's own, more careful conclusion is **Unknown, flagged as a concerning signal**, not a confirmed fail. A Result that only stores PASS/FAIL/UNKNOWN as a formula output would either lose this nuance or need special-cased exceptions per row. Review already solves this exact problem for qualitative criteria (a documented 1–5 conclusion backed by Reason/Confidence, informed by but not mechanically dictated by Evidence) — the Hard Requirement Result should follow the same pattern, not Scoring's.

---

# Current Assessment

**Expressed Correctly in Intent, Inconsistently in Practice.**

`docs/03_scoring-model.md` is unambiguous that Hard Requirements gate entry to the pipeline ("Configurations failing one or more Hard Requirements shall not receive an Overall Score"), and the live data shows real cases where this was followed correctly — Renault's `EVOLUTION_URBAN`/`EVOLUTION_COMFORT` were excluded from `10_Scoring` entirely for failing M004, confirmed by zero Score rows for either Configuration.

It is not "Correctly Enforced" because the mechanism is manual, and manual re-derivation from prose has already produced gaps and one confirmed miss:

* **Tesla's M002 failure was not caught until 8+ criteria into scoring** (`DEC_M002_VIOLATION`), specifically because nothing runs the Hard Requirement check before Reviews begin — the check only happened by accident, while a different criterion (Garasje, W019) needed the same Length figure.
* **3 of Tesla's 4 Configurations have zero `05_Equipment` rows for `EQ_ACC`/`EQ_REAR_CAMERA`** (only `TESLA_MODEL_3_LONG_RANGE_RWD` does), so M004/M005 status for `TESLA_MODEL_3_RWD`, `LONG_RANGE_AWD`, and `PERFORMANCE` is genuinely unrecorded — yet their Configuration Notes say nothing about M004 or M005 at all, silent rather than explicitly flagged Unknown.
* **2 of Volvo's 3 Configurations have the same gap** for both requirements; the third (`P8_AWD`) has `EQ_ACC` confirmed but still no `EQ_REAR_CAMERA` row. Volvo's Notes extend the P8 AWD's confirmed ACC fact to the other two Configurations by prose inference ("same driver-assistance suite across the range") rather than a recorded fact per Configuration.
* **Kia already has both facts available** (`EQ_ACC`/`EQ_REAR_CAMERA` both `STANDARD`) but its Configuration Notes never states M004/M005 compliance at all — the answer exists in the workbook and was simply never surfaced to the Hard Requirement question it also answers.

---

# Hard Requirement Compliance Matrix (mechanically derived for this audit, not stored anywhere today)

| Configuration | M002 (Length) | M003 (Winter range) | M004 (ACC) | M005 (Rear camera) |
| --- | --- | --- | --- | --- |
| TESLA_MODEL_3_RWD | FAIL | UNKNOWN | UNKNOWN | UNKNOWN |
| TESLA_MODEL_3_LONG_RANGE_AWD | FAIL | UNKNOWN | UNKNOWN | UNKNOWN |
| TESLA_MODEL_3_LONG_RANGE_RWD | FAIL | UNKNOWN | PASS | PASS |
| TESLA_MODEL_3_PERFORMANCE | FAIL | UNKNOWN | UNKNOWN | UNKNOWN |
| RENAULT_4_EVOLUTION_URBAN | PASS | UNKNOWN | FAIL | PASS |
| RENAULT_4_EVOLUTION_COMFORT | PASS | UNKNOWN | FAIL | PASS |
| RENAULT_4_TECHNO | PASS | UNKNOWN* | PASS | PASS |
| RENAULT_4_TECHNO_PLEIN_SUD | PASS | UNKNOWN | PASS | PASS |
| RENAULT_4_ICONIC | PASS | UNKNOWN | PASS | PASS |
| RENAULT_4_ICONIC_PLEIN_SUD | PASS | UNKNOWN | PASS | PASS |
| SKODA_ELROQ_ESSENCE_60 | PASS | UNKNOWN | PASS | PASS |
| SKODA_ELROQ_SELECTION_60 | PASS | UNKNOWN | PASS | PASS |
| SKODA_ELROQ_SELECTION_85X | PASS | UNKNOWN | PASS | PASS |
| SKODA_ELROQ_RS | PASS | UNKNOWN | PASS | PASS |
| VOLVO_EX30_P5 | PASS | UNKNOWN | UNKNOWN | UNKNOWN |
| VOLVO_EX30_P5_LONG_RANGE | PASS | UNKNOWN | UNKNOWN | UNKNOWN |
| VOLVO_EX30_P8_AWD | PASS | PASS | PASS | UNKNOWN |
| KIA_EV2_FWD_LONG_RANGE_GT_LINE | PASS | UNKNOWN | PASS | PASS |

\* A naive `≥300km` comparison against Renault 4 Techno's only recorded winter figure (287km) computes FAIL. The correct answer, per the framework's own existing prose caveat, is **UNKNOWN with a concerning proxy signal** — the 287km point is from a different trim tested in mild (5–7°C), not winter, conditions. This is the concrete case demonstrating why Result cannot be a pure threshold formula (see Executive Summary).

M001 (BEV) omitted — no structured field exists, and every Configuration in this workbook passes by curation; not a real per-row risk today.

---

# Structural Options Considered

| Option | Recommendation | Rationale |
| --- | --- | --- |
| Status quo: Notes prose + `11_DecisionLog` cross-references | **Reject** | Already produced one confirmed miss (Tesla M002) and multiple silent gaps (3/4 Tesla and 2/3 Volvo Configurations with no recorded M004/M005 answer despite partial Equipment data existing). Nothing catches a missing check; nothing makes the matrix above queryable without reading every Configuration's Notes end to end. |
| Pure calculated worksheet: `Result` as a formula over `04_Technical`/`05_Equipment` thresholds, mirroring `10_Scoring.RawScore` | **Reject** | The Renault 4 Techno M003 case shows a naive threshold formula produces a confidently wrong answer (FAIL) where the correct answer is UNKNOWN-with-caveat. Hard Requirement evidence quality varies (mismatched trims, proxy conditions, unverified assumptions) in exactly the way Review already exists to capture for qualitative criteria — a pure formula can't represent "the number crossed the line, but the evidence backing that number is weak." |
| Contributor-interpreted result entity (mirrors `Review`): PASS/FAIL/UNKNOWN + Reason + Confidence, referencing the supporting Technical/Equipment/Evidence | **Adopt** | Directly implements the proposed `Configuration → Hard Requirement Result → PASS/FAIL/UNKNOWN → Reason` shape, with Confidence added so a contributor can express "this crosses the threshold, but the evidence is weak" the same way Review already does for every other criterion. One row per Configuration × Hard-Requirement-Criterion makes the matrix above a real, queryable worksheet instead of a one-off audit script. |

---

# Risks

**Risk of adopting the Result entity now:**

* A new worksheet plus backfilling 18 Configurations × 5 Hard Requirements (90 rows) is real migration work, not free — though most cells can be seeded directly from `04_Technical`/`05_Equipment` (as this audit's matrix shows) rather than re-researched from scratch.
* Deciding the PASS/FAIL/UNKNOWN threshold for each Hard Requirement (e.g., is 287km at the wrong trim/conditions UNKNOWN or a low-confidence FAIL?) requires a judgment call per requirement type — this audit recommends UNKNOWN-with-Reason for proxy/mismatched evidence, but a future contributor could reasonably argue differently; worth settling explicitly in the follow-up decision rather than leaving implicit.
* This does not yet solve the separate, previously-flagged question of how a **confirmed FAIL kept for comparison by explicit user override** (Tesla M002) should be represented — that is a Configuration-eligibility decision, not a property of the Result row itself, and should be resolved in the same follow-up rather than deferred again.

**Risk of leaving Hard Requirement compliance as Notes prose:**

* The exact failure mode that already happened once (Tesla M002 undetected for 8+ criteria) will recur as more vehicles are added — nothing runs the check before Reviews begin, so catching it remains dependent on some other criterion accidentally needing the same fact.
* Facts that already exist in the workbook (Kia's M004/M005 answers, sitting unused in `05_Equipment`) will keep going unsurfaced, and gaps (3/4 Tesla, 2/3 Volvo Configurations missing M004/M005 data entirely) will keep being silent rather than flagged, since nothing currently requires every Configuration × Hard-Requirement pair to have an explicit answer.

---

# Recommendation

**Adopt a Hard Requirement Result entity, structured like Review rather than like Scoring.**

Proposed shape, for a follow-up ADR:

```text
HardRequirementResultID, ConfigurationID, CriterionID, Result, Reason, Confidence, EvidenceID, FrameworkVersion
```

* `CriterionID` references a `01_Criteria` row where `Type = HARD` (already a real, existing distinction — no new Criterion concept needed).
* `Result` is a new enumeration: `PASS`, `FAIL`, `UNKNOWN`.
* `Reason` and `Confidence` follow the same pattern as `Review` — a contributor's interpretation of the supporting Technical/Equipment/Evidence, not a formula. Where a raw fact exists (`04_Technical`, `05_Equipment`) it should inform the Result but not silently override a contributor's documented caveat (per the Renault 4 Techno case).
* One row per Configuration × Hard-Requirement-Criterion. `CoveragePercent`-style completeness reporting (mirroring `12_OverallScores`) becomes possible once every Configuration has all five rows populated.

Separately, resolve how a confirmed FAIL that is deliberately kept for comparison (Tesla M002) should be represented — either a distinct `Configuration` status value extending ADR-004's pattern, or a dedicated override flag alongside the Result row — in the same follow-up decision, since this audit surfaced it but does not resolve it on its own.

This should proceed as a follow-up ADR (new worksheet, new enumeration, likely Framework Version increment given it changes how the Stage-1 gate is enforced) — the same two-step pattern used for ARCH-007/ADR-007 and ARCH-008/ADR-008.

---

# Definition of Done

* [x] Current Hard Requirement handling evaluated against `docs/02_criteria-and-weighting.md`, `docs/03_scoring-model.md` Stage 1, `entity-model.md`, `workbook-schema.md`, ADR-004, and the live Reference Workbook.
* [x] All 18 live Configurations checked against all 5 Hard Requirement criteria (M001–M005) by cross-referencing `04_Technical` and `05_Equipment`, producing a full compliance matrix.
* [x] Confirmed zero `10_Scoring` rows exist for Hard-Requirement criteria (correctly excluded from weighted scoring).
* [x] Identified a concrete case (Renault 4 Techno, M003) demonstrating why Result cannot be a pure calculated formula.
* [x] Structural options assessed against observed data, not hypotheticals.
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `entity-model.md`, `workbook-schema.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: adopt a Review-shaped Hard Requirement Result entity via a follow-up ADR; resolve the FAIL-but-kept-for-comparison override question in the same ADR.
