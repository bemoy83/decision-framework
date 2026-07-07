# ARCH-011 — Notes Strategy Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.7
**Related:** workbook-schema.md Data Entry Rules, ADR-006, ADR-008, ADR-009, ARCH-007, ARCH-009, ARCH-010

---

# Question Being Answered

`Notes` fields exist on nearly every worksheet, documented only as "optional notes," with no rule for what they may or may not contain. Should the framework define an explicit allow-list (short explanation, deviation reference, implementation comment) and deny-list (review, evidence, history, decision log) for what `Notes` may hold?

---

# Executive Summary

**Yes — and the framework already half-agrees with itself.** `workbook-schema.md`'s Data Entry Rules already state "Never duplicate information owned by another worksheet" and "Notes are intended for implementation comments, clarification and data collection context," but neither rule is operationalized: nothing defines what counts as duplication, and nothing has ever checked whether a given `Notes` cell honors it. The live data shows it often doesn't.

The clearest evidence is not hypothetical — it is the framework's own recent work catching up with itself. `03_Configurations.Notes` for the four Tesla Configurations contains a full paragraph narrating the M002 length violation (discovery, override rationale, "kept for comparison at the user's request") and a second paragraph narrating the M003 winter-range investigation (which tests were found, why neither matches, why it remains Unknown). Both paragraphs are now **fully duplicated** by structured data introduced during this same review: `14_HardRequirementResults` (ADR-009) already records M002 as FAIL and M003 as UNKNOWN with the identical reasoning in its `Reason` field, `Configuration.HardRequirementOverride = TRUE` already records the override, and `11_DecisionLog`'s `DEC_M002_VIOLATION` already records the discovery narrative. The Notes prose was the *only* home this information had when it was written; it is not the only home anymore, and nobody went back to retire it once the real home was built (ADR-009 itself flagged this gap under "Alternatives Considered" for `12_OverallScores` specifically, but the same gap exists in `03_Configurations.Notes`, more broadly).

`02_Vehicles.Notes` shows a related but distinct problem: every one of the 5 rows is 100% Evidence/Source-provenance text ("[Manufacturer] page reviewed [date]: [URL]; [what was/wasn't found]") — squarely the job of `09_Sources`/`08_Evidence`, never rewritten into Vehicle-level "explanation." By contrast, `09_Sources.Notes` (all 87 rows populated) and `06_EquipmentDefinitions.Notes` show the pattern working correctly: short, single-purpose text describing what a Source covers or how a Definition is scored, never drifting into Review or Evidence territory. The difference between the good and bad cases is not the *column*, it is the *worksheet*: Reference worksheets (Sources, EquipmentDefinitions) stay disciplined; Operational worksheets that accumulate content across many sessions (Configurations, Vehicles) do not.

---

# Current Assessment

**Principle Present, Never Operationalized.**

`workbook-schema.md` already states the right instinct — Notes should not duplicate another worksheet's information, and should stay to implementation comments/clarification/data-collection context — but "duplicate" and "clarification" are both undefined enough that a contributor cannot check their own Notes cell against the rule before writing it. This is the same shape of gap ARCH-010 found for Framework Versioning: the right principle stated in prose, never made into something checkable.

| Worksheet | Notes content observed | Verdict |
| --- | --- | --- |
| `09_Sources` (87/87 rows) | "What this source covers" (e.g. "Dekker fjæring/komfort, sikt og infotainment-betjening") | **Correct usage** — short, scoped, never restates a Review's conclusion or an Evidence observation itself. |
| `06_EquipmentDefinitions` | One-line scoring-treatment comments (e.g. "Må-krav, men kvalitet scores separat.") | **Correct usage** — implementation comment, exactly the allowed category. |
| `02_Vehicles` (5/5 rows) | 100% Evidence/Source-provenance narration: "[page] reviewed [date]: [URL]; [field] not specified in workbook-supported source fields." | **Wrong worksheet.** This is `08_Evidence`/`09_Sources`'s job; none of it is "explanation," all of it is "documented observation." |
| `03_Configurations` (Tesla, 4/18 rows) | Hard Requirement violation narrative, override rationale, winter-range investigation writeup | **Duplicate.** Fully superseded by `14_HardRequirementResults` + `Configuration.HardRequirementOverride` + `11_DecisionLog.DEC_M002_VIOLATION`, none of which existed when the prose was written. |
| `12_OverallScores` (Tesla row) | Hardcoded formula string: `"HARD REQUIREMENT FAILED (M002...) - NOT ELIGIBLE, kept only for user comparison."` | **Duplicate, already tracked.** ADR-009 explicitly deferred rewiring this to derive from `HardRequirementResult`/`HardRequirementOverride` automatically — same root cause as the `03_Configurations` finding above. |

---

# What Belongs, What Doesn't

Adopting the user's proposed categories, refined slightly with a third allowed form (a pointer, distinct from a restatement):

**Allowed:**
* Short explanation of something not otherwise self-evident from structured columns (e.g. `09_Sources`'s "what this source covers").
* A **pointer** to where a deviation or decision is recorded — "see `14_HardRequirementResults` for M002/M003 status; see `DEC_M002_VIOLATION`" — not a restatement of that record's content.
* Implementation comment (a scoring-treatment note, a data-entry caveat, e.g. "confirmed via search-result synthesis, not a direct fetch").

**Not allowed:**
* Review — an interpretation of what evidence means belongs in `07_Reviews`.
* Evidence — a documented observation with its own source/date belongs in `08_Evidence` or `09_Sources`.
* History — a narrative of what happened and when belongs in `11_DecisionLog`.
* Decision rationale — belongs in `11_DecisionLog`, referenced by pointer, not restated.

This maps directly onto the existing but under-specified Data Entry Rule #3 ("never duplicate information owned by another worksheet") — the allow-list is what's left over once everything with its own structural home is pointed to instead of repeated.

---

# Risks

**Risk of ratifying and applying this now:**

* Pruning `03_Configurations.Notes` for the 4 Tesla rows is low-risk: every fact removed is independently verifiable in `14_HardRequirementResults`/`11_DecisionLog`, so nothing is lost, only de-duplicated.
* Pruning `02_Vehicles.Notes` (100% Evidence-shaped across all 5 rows) is a larger, different kind of task — the content there was never given a structured home in the first place (no corresponding `08_Evidence` rows exist for most of it), so "pointing instead of restating" would require creating new Evidence rows, not just deleting prose. That is a data-gathering task, not a housekeeping edit, and is better scoped as its own follow-up than bundled here.
* Rewiring `12_OverallScores.Notes`'s hardcoded string is already identified and deliberately deferred in ADR-009 for a specific reason (it touches ADR-005-governed formula logic across all scored Configurations) — this audit does not reopen that scoping decision.

**Risk of leaving it unratified:**

* The exact failure mode this audit demonstrates — new structure gets built, but the prose the structure was meant to replace never gets retired — will recur every time a future ARCH/ADR closes a P3/P6-style gap, since nothing currently prompts anyone to go back and prune. Notes accumulates monotonically; nothing today ever shrinks it.

---

# Recommendation

**Ratify the allow/deny policy as a documented rule; apply it to the one case where pruning is fully safe today (Tesla's `03_Configurations.Notes`); defer the two cases that require new data-gathering or already-scoped formula work rather than a housekeeping edit.**

1. Expand `workbook-schema.md`'s Data Entry Rule #8 into the explicit allow/deny list above, replacing the current single vague sentence.
2. Prune `03_Configurations.Notes` for the 4 Tesla Configurations: remove the M002/M003 narrative paragraphs (now fully duplicated by `14_HardRequirementResults` and `11_DecisionLog`), replace with a short pointer sentence.
3. Explicitly defer, not silently drop: `02_Vehicles.Notes`'s Evidence-shaped content (needs real `08_Evidence` population before it can be pointed-to instead of restated — a data task, out of scope here) and `12_OverallScores.Notes`'s hardcoded string (already scoped as deferred work in ADR-009).

This can proceed as a lightweight follow-up ADR (documentation rule + the one safe pruning example), the same two-step pattern used for P1–P4.

---

# Definition of Done

* [x] Every populated Notes field surveyed for content type: `02_Vehicles` (5/5), `03_Configurations` (18/18), `09_Sources` (87/87), `06_EquipmentDefinitions`, `12_OverallScores` (5/5).
* [x] Identified the existing but unoperationalized principle already in `workbook-schema.md` (Data Entry Rules #3 and #8) that this audit's rule directly implements.
* [x] Found and quantified concrete duplication: Tesla's `03_Configurations.Notes` fully superseded by `14_HardRequirementResults`/`11_DecisionLog`, `12_OverallScores.Notes` already flagged as deferred in ADR-009.
* [x] Distinguished a systemic, harder problem (`02_Vehicles.Notes` being 100% Evidence-shaped with no structured home to point to yet) from an easy one (Tesla's Configuration Notes, fully safe to prune).
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `workbook-schema.md` and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: ratify the allow/deny rule, apply it to the one safe case now, defer the two cases that require more than housekeeping.
