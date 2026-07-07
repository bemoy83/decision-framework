# ADR-011 — Notes Strategy Policy

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.7 (unchanged)

**Supersedes:** None

**Related:**

* ADR-009 — Hard Requirement Result Model
* ADR-010 — Framework Versioning Rule
* audits/ARCH-011-Notes-Strategy-Evaluation.md
* framework/architecture/workbook-schema.md

---

# Context

ARCH-011 evaluated whether `Notes` fields — present on nearly every worksheet, documented only as "optional notes" — needed an explicit rule for what they may contain. It found `workbook-schema.md`'s Data Entry Rules already state the right instinct (Rule 3: "never duplicate information owned by another worksheet"; Rule 8: "Notes are intended for implementation comments, clarification and data collection context") without ever defining "duplicate" or "clarification" precisely enough to check a Notes cell against.

The audit found concrete, current duplication caused by this gap: `03_Configurations.Notes` for all four Tesla Configurations narrates the M002 length violation and (for `TESLA_MODEL_3_LONG_RANGE_RWD`) the M003 winter-range investigation in full prose. Both are now fully duplicated by structure introduced earlier in this same review — `14_HardRequirementResults` (ADR-009) records identical PASS/FAIL/UNKNOWN conclusions with matching `Reason` text, `Configuration.HardRequirementOverride` records the override, and `11_DecisionLog`'s `DEC_M002_VIOLATION` records the discovery and override narrative in full. None of that structure existed when the Notes prose was written; nobody returned to retire the prose once it did.

The audit also found two harder cases that are the same root problem but are not safely fixable by pruning alone: `02_Vehicles.Notes` (100% Evidence/Source-provenance text with no corresponding `08_Evidence` rows to point to instead) and `12_OverallScores.Notes` (a hardcoded formula string for Tesla, already explicitly deferred as future work in ADR-009). Both are named here but intentionally out of scope for this ADR.

---

# Decision

`workbook-schema.md`'s Data Entry Rule 8 is replaced with an explicit allow/deny list for `Notes` content:

**Allowed:**
* Short explanation of something not otherwise self-evident from structured columns.
* A pointer to where a deviation or decision is recorded (e.g. "see `14_HardRequirementResults`; see `DEC_M002_VIOLATION`") — never a restatement of that record's content.
* Implementation comment (a scoring-treatment note, a data-entry caveat).

**Not allowed:**
* Review — an interpretation of evidence belongs in `07_Reviews`.
* Evidence — a documented observation with its own source/date belongs in `08_Evidence` or `09_Sources`.
* History — a narrative of what happened and when belongs in `11_DecisionLog`.
* Decision rationale — belongs in `11_DecisionLog`, referenced by pointer, not restated.

`03_Configurations.Notes` for the four Tesla Configurations is pruned: the M002 violation narrative and (for `TESLA_MODEL_3_LONG_RANGE_RWD`) the M003 investigation narrative are replaced with a short pointer to `14_HardRequirementResults` and `DEC_M002_VIOLATION`. Sentences that have no other structured home today — the Tesla page review/price-sourcing sentence and the Configuration Status rationale sentence — are left untouched, since `03_Configurations` has no `SourceID` column and pruning them would discard information with nowhere else to live.

`02_Vehicles.Notes` and `12_OverallScores.Notes` are explicitly not touched by this ADR. Fixing them properly requires either populating real `08_Evidence` rows (a data-gathering task) or rewiring `12_OverallScores`' formula logic (already scoped and deferred in ADR-009) — neither is a housekeeping edit, and bundling either here would blur a disciplined, low-risk change with a much larger one.

---

# Rationale

The rule is not new policy so much as a precise version of what `workbook-schema.md` already asserted. Stating "pointer, not restatement" as the dividing line gives a contributor a mechanical test: if the same fact could be looked up in another worksheet, Notes should say where, not say what.

Pruning only the Tesla case (not `02_Vehicles` or `12_OverallScores`) follows the same discipline ARCH-009 and ARCH-010 both used: fix what's safely fixable now, name what isn't, and let a future, appropriately-scoped pass handle it rather than forcing every related problem into one change.

---

# Consequences

## Workbook Schema

`framework/architecture/workbook-schema.md`'s Data Entry Rules gain the explicit allow/deny list in place of the current single sentence.

## Reference Workbook

* `03_Configurations.Notes` reworded for `TESLA_MODEL_3_RWD`, `TESLA_MODEL_3_LONG_RANGE_AWD`, `TESLA_MODEL_3_LONG_RANGE_RWD`, and `TESLA_MODEL_3_PERFORMANCE` — M002/M003 narrative replaced with a pointer; all other content (page-review/price-sourcing, Configuration Status rationale) unchanged.
* `11_DecisionLog` gains an entry recording this change.

No Framework Version increment — this is a documentation-rule and wording change, the same class of change ADR-008 established does not require one.

---

# Alternatives Considered

## Prune all 18 Configurations' Notes and all 5 Vehicles' Notes in one pass

Rejected. Only the Tesla case has a fully safe replacement today (the removed content is independently verifiable elsewhere). The other 14 Configurations' Notes were not found to contain the same kind of now-duplicated content during this audit, and `02_Vehicles.Notes` would require new Evidence rows, not just deletion, to prune safely — a different and larger task.

## Rewire `12_OverallScores.Notes` now, since it's the same root cause

Rejected. Already explicitly scoped and deferred in ADR-009 for a specific reason (it touches ADR-005-governed formula logic across all scored Configurations); reopening that scoping decision here would undercut the discipline that deferral was meant to preserve.

## Leave the principle as general prose, without an explicit allow/deny list

Rejected. This is the status quo ARCH-011 found insufficient — "clarification" and "duplicate" have no operational test today, which is exactly how the Tesla duplication went unnoticed.

---

# Migration Strategy

No historical data changes meaning. The pruned Tesla Notes text is reformatted, not reinterpreted — every fact removed remains independently recorded and verifiable in `14_HardRequirementResults` and `11_DecisionLog`.

---

# Impact

Affected documents:

* framework/architecture/workbook-schema.md

Affected workbook:

* `03_Configurations` (Notes reworded for 4 Tesla rows)
* `11_DecisionLog` (new entry)

No changes are required to:

* `02_Vehicles`, `12_OverallScores` (explicitly deferred, see Alternatives Considered)
* Any other Configuration's Notes (not found to contain now-duplicated content)

---

# Guiding Principle

> **Notes should say where to look, not repeat what's already written down. The moment a fact gets a real structural home, the prose that stood in for it becomes debt, not documentation.**
