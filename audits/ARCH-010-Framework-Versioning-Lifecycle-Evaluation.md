# ARCH-010 — Framework Versioning Lifecycle Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.6
**Related:** ADR-004, ADR-005, ADR-006, ADR-007, ADR-008, ADR-009, docs/02_criteria-and-weighting.md, docs/03_scoring-model.md, entity-model.md, 11_DecisionLog

---

# Question Being Answered

`FrameworkVersion` and `WorkbookVersion` work today. Before more vehicles are added, what actually happens when Framework moves from one version to the next? Must every Configuration be re-scored? Can the same dataset legitimately exist under more than one Framework Version at once? The user's own hypothesis going in: the model does not need to change, only the rules need to be written down clearly.

---

# Executive Summary

**The hypothesis is confirmed: no model change is needed. The workbook already has every piece required to answer all three questions — it has simply never written down the rule that makes the answer mechanical instead of a judgment call each time.**

The evidence is the framework's own history, reconstructed in full from `11_DecisionLog`: every one of the seven times `FrameworkVersion` has actually incremented (0.2→1.0→1.2→1.3→1.4→1.5→1.6), the trigger was a **schema change** (a new/changed worksheet, column, or enumeration) — not a weighting change. Not a single one of the framework's 29 Criteria has ever changed its `Weight`, `Type`, or `HardRequirement` flag since the workbook's inception (all 29 rows still carry `FrameworkVersion = 1.0`). The one exception, ADR-005, introduced the scoring *calculation engine* itself (a one-time bootstrapping event, not a revision of an existing rule) — every other bump, including all three from this session's own P1–P3 work (ADR-007, ADR-008, ADR-009), added or corrected structure without touching any existing Criterion's weight or the `RawScore`/`WeightedScore` formula.

This distinction — **schema bump** vs. **methodology bump** — is the rule the docs gesture at ("every significant methodology change creates a new Framework Version") but never define. Once stated explicitly, all three of the user's questions answer directly from data already in the workbook, with no new column or worksheet required.

---

# The Version Bump Ledger (reconstructed from `11_DecisionLog`)

| Transition | Cause | Type |
| --- | --- | --- |
| 0.2 → 1.0 | Synchronized workbook structure with Framework Version 1.0 | Schema |
| 1.0 → 1.2 | ADR-004: Configuration Status column added | Schema |
| 1.2 → 1.3 | ADR-006: Evidence schema reconciliation (column merge/removal) | Schema |
| 1.3 → 1.4 | ADR-005: Scoring calculation engine introduced (RawScore/WeightedScore formulas, OverallScore worksheet) | Methodology (bootstrapping) |
| 1.4 → 1.5 | ADR-007: Technical Field Catalog (new worksheet + column) | Schema |
| 1.5 → 1.6 | ADR-009: Hard Requirement Result model (new worksheet + column) | Schema |

Not shown as a transition because it correctly did **not** bump the version: ADR-008 (Review wording correction) — no structural or weighting change, `FrameworkVersion` stayed at 1.5.

Also notable: a large amount of real work happened at `FrameworkVersion 1.4` with no `11_DecisionLog` entry at all for individual scoring rounds (e.g. the W010–W013 and W018 criterion rounds, the Kia W014 revision, the Skoda/Volvo W018 reasoning revision) — `11_DecisionLog`'s own Notes describe it as documenting "significant" changes, and in practice routine scoring rounds using the existing methodology have never been logged there, only framework-structural events and a few flagged discoveries (e.g. `DEC_M002_VIOLATION`). This is consistent with the rule below, not a violation of it, but worth naming so a future contributor does not expect `11_DecisionLog` to be a complete activity log.

---

# Live Evidence That Mixed-Version Coexistence Already Works

Today, right now, in the live workbook:

* `07_Reviews`: 7 rows carry `FrameworkVersion = 1.2`, 93 rows carry `1.4`.
* `10_Scoring`: all 100 rows carry `1.4` — including the 7 Score rows whose linked Review is still stamped `1.2`.
* `12_OverallScores`: all 5 rows carry `1.4`, even though the workbook's active version is now `1.6`.

This is not an inconsistency to fix — it is the answer to "can the same dataset exist under multiple Framework Versions" demonstrated in production. The 7 original vertical-slice Reviews were authored under `1.2`'s criteria definitions and have never needed rewriting; when ADR-005 (`1.4`) later formalized *how* their Score is calculated, the **Score** row was correctly re-stamped `1.4` (the ruleset that now governs its calculation), while the **Review** row correctly kept `1.2` (the qualitative content itself is unchanged and was authored under criteria that also haven't changed). A row's `FrameworkVersion` means "the ruleset that produced this row's own value," not "when the underlying facts were gathered" — a distinction the docs imply but never state.

By the same logic, the 5 existing `12_OverallScores` rows (still `1.4`) do not need regeneration today even though the active version is `1.6`, because neither ADR-007 nor ADR-009 changed any Criterion's `Weight` or the `RawScore`/`WeightedScore` formula those rows depend on — both were schema bumps. They would need regeneration only if a future bump changes something they actually depend on.

---

# The Rule

**A `FrameworkVersion` increment is triggered by exactly one of two kinds of change, and the required response differs by kind:**

**1. Schema change** — a new/changed/removed worksheet or column, a new enumeration, or a documentation correction that does not alter any existing Criterion's `Weight`, `Type`, `HardRequirement` flag, or the `RawScore`/`WeightedScore` calculation rule.
→ *Consequence:* No existing Review, Score, or OverallScore becomes invalid or needs regeneration. They remain fully comparable to evaluations produced after the bump, because nothing about how they are computed has changed. The version still increments, so the workbook's schema shape at any point in time remains traceable via `11_DecisionLog`.

**2. Methodology change** — any change to a Criterion's `Weight`, `Type`, `HardRequirement` flag, Active status, or to the `RawScore`/`WeightedScore` calculation rule itself, or to a Hard Requirement's own pass/fail threshold.
→ *Consequence:* Any already-computed Score for the *affected Criterion* is no longer comparable to a Score for that same Criterion computed after the bump (per docs/02's "Scores generated using different Framework Versions shall never be compared directly"). Only the affected Criteria need re-scoring to restore comparability — not necessarily every Criterion for every Configuration, and not necessarily every Configuration.

**To answer "does an existing evaluation need re-scoring after a bump":** scan `11_DecisionLog` for any Methodology-type entry between the evaluation's own `FrameworkVersion` (exclusive) and the workbook's current `FrameworkVersion` (inclusive). If none exists — as is true for every Configuration's existing evaluations today, since 1.4→1.5→1.6 were both Schema bumps — no re-scoring is needed. If one exists, only the Criteria it names need re-scoring.

---

# Risks

**Risk of leaving this undocumented:**

* The next genuine methodology change (a real weight revision, or a Hard Requirement threshold change) will raise exactly the question this audit answers, without a documented rule to answer it by — likely re-litigated from scratch, as this topic itself was flagged for exactly that reason.
* A future contributor could reasonably read "every significant methodology change creates a new Framework Version" as license to bump the version for *any* change (schema included) or, conversely, avoid bumping it for a real weighting change by rationalizing it as "not that significant" — both misreadings are avoidable with the two-kind rule stated explicitly.

**Risk of documenting the rule as proposed:**

* Minimal. This is a documentation exercise describing behavior the workbook's own history already demonstrates; it does not require any new column, worksheet, or migration.

---

# Recommendation

**No model change required. Document the rule.**

1. Add the Schema-vs-Methodology distinction and its consequence (full text above) to `docs/02_criteria-and-weighting.md`'s Versioning section and `docs/03_scoring-model.md`'s Framework Integrity section, replacing the current single undefined line ("every significant methodology change creates a new Framework Version") with the two-kind rule and its regeneration consequence.
2. Add a short note to `entity-model.md`'s `FrameworkVersion` entity clarifying that a row's `FrameworkVersion` records the ruleset that produced *that row's own value*, not necessarily when its underlying facts were gathered — using the existing Review-1.2/Score-1.4 coexistence as the documented example.
3. Optional, low-risk, not required by the rule itself: add a `VersionBumpType` column (`SCHEMA` / `METHODOLOGY`) to `11_DecisionLog`, populated retroactively for the six existing transitions per the ledger above, so the "scan DecisionLog" step in the rule becomes a filter instead of a prose read. This is the only candidate schema touch in this audit, and it is genuinely optional — the rule works today by reading Decision text manually, exactly as this audit just did.

This can proceed as a documentation-only follow-up (items 1–2), with item 3 as a separate, explicitly optional minor enhancement the user can decline without weakening the rule itself.

---

# Definition of Done

* [x] Reconstructed the complete Framework Version bump history from `11_DecisionLog` (7 transitions, including this session's own ADR-007/008/009).
* [x] Confirmed no Criterion's Weight/Type/HardRequirement flag has changed since inception (all 29 rows still `FrameworkVersion 1.0`).
* [x] Confirmed live mixed-version coexistence already exists in production data (`07_Reviews` 1.2/1.4 split) and explained why it is correct, not a defect.
* [x] Confirmed the three most recent bumps from this session's own P1–P3 work (1.4→1.5→1.6) required no re-scoring, and explained why via the proposed rule.
* [x] Answered all three of the user's original questions directly: what happens at a version bump (depends on Schema vs. Methodology), whether re-scoring is required (only for Methodology bumps, and only for the affected Criteria), and whether the same dataset can exist under multiple versions (yes, already does, by design).
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `docs/02_criteria-and-weighting.md`, `docs/03_scoring-model.md`, `entity-model.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: document the rule in two existing docs; one optional, non-required minor enhancement (`11_DecisionLog.VersionBumpType`) offered separately.
