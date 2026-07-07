# ARCH-007 — Technical Entity: Property Bag Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.4
**Related:** ADR-003, ADR-006, entity-model.md, workbook-schema.md, implementation-contract.md, relationships.md, ARCH-006

---

# Question Being Answered

Is `04_Technical` a property bag (entity-attribute-value model), or a table with fixed columns per characteristic — and does the current shape scale to ~50 vehicles / 2,000–5,000 technical data points without duplication or special cases?

---

# Executive Summary

`04_Technical` already **is** a property bag, and has been since Framework Version 1.0 — `entity-model.md`, `workbook-schema.md`, and ADR-003 all describe it as one row per (Vehicle-or-Configuration, property, value) fact, not as a wide table. That choice is correct for this domain and should not change: vehicles have a long tail of sparse, model-specific attributes (frunk volume, trailer weight, wheel-size-dependent range) that would force an ever-growing set of mostly-NULL columns under a fixed-column design. The live data confirms this — of 22 distinct properties recorded across 5 vehicles, several apply to exactly one vehicle.

The real finding is that today's property bag is **half-built**. `05_Equipment` solved the exact same shape of problem — many heterogeneous facts, one fact table — by pairing it with `06_EquipmentDefinitions`, a Reference worksheet that owns the canonical vocabulary (`EquipmentDefinitionID`, `Name`, `Category`, `Description`), while `Equipment` itself only stores `EquipmentDefinitionID` + `Availability`. `Technical` has no equivalent catalog. Its property name (`TechnicalField`) is raw, unvalidated free text, invented ad hoc per row with no reference table to check it against.

At 5 vehicles / 145 real rows, this has already produced exactly the kind of drift a catalog would prevent:

* **"Kerb weight" vs "Kerb weight (MRO)"** — Tesla's four configurations use the `(MRO)` variant exclusively; Renault and Volvo's seven configurations use the plain variant exclusively. The two labels never co-occur on the same vehicle, so there is no evidence this is a deliberate framework distinction — it reads as two contributors independently inventing a name for a manufacturer-specific measurement convention. A cross-vehicle "kerb weight" query must already know to check two strings.
* **"Width" vs "Width (mirrors folded)" vs "Width (mirrors unfolded)"** — Tesla records both mirror states explicitly (Vehicle-level unfolded, Configuration-level folded); Škoda and Volvo each record a single unqualified `Width` with no way to know which state it reflects. Width is not safely comparable across manufacturers today.
* **"WLTP range" vs "WLTP range (18\" wheels)" / "WLTP range (19\" wheels)"** — `TESLA_MODEL_3_LONG_RANGE_RWD` has two range rows distinguished only by a parenthetical suffix on the property name, because ADR-003 explicitly anticipates "option-dependent specifications" as Configuration-level Technical facts but never specifies *how* a sub-option should be represented. This is the same class of problem already surfaced for P1 in the Renault boot-volume trim split, in a third distinct shape: there it was one fact varying across sibling Configurations of a Vehicle; here it's one fact varying across sub-options *within a single Configuration*.
* **Six orphaned rows** with `TechnicalID`, `VehicleID`, `ConfigurationID`, `SourceID`, and `Value` all blank (only `TechnicalField`, `Unit`, and `Confidence=UNKNOWN` populated, `FrameworkVersion=1.0`) — these directly violate `implementation-contract.md`'s mandatory rule that "Technical references either one Vehicle or one Configuration." Nothing in the framework catches this; they have apparently sat unnoticed since v1.0.
* **Documentation/implementation drift**: `workbook-schema.md` documents the property-name column as `Property` (9 columns total); the live workbook calls it `TechnicalField` and carries a 10th column, `FrameworkVersion`, that isn't documented at all. Under ADR-001 ("Documentation Is Source of Truth"), this is a live contract violation independent of the EAV question.

None of this is damaging at today's scale — five vehicles is small enough to eyeball. It is exactly the failure mode the user is asking about *before* it stops being eyeball-able: nothing in the schema stops a 6th contributor from typing a third naming convention for kerb weight, or a fourth for width, and at 50 vehicles / thousands of rows that drift compounds silently, one contributor at a time, with no query able to reliably group "the same fact" together.

---

# Current Assessment

**Adequately Structured, Incompletely Governed.**

The Vehicle/Configuration-referencing fact-table half of the property bag is sound: `TechnicalID` is a stable, unique identifier (145/145 populated rows verified unique in the live data); the "belongs to Vehicle XOR Configuration" rule is the right invariant for the reasons already established in ARCH-006 (Vehicle-level facts shared across configs, Configuration-level facts that differ per trim). Sparse, model-specific attributes (`Frunk volume`, `Max trailer weight` — Kia-only in the live data) are handled cleanly, with no forced NULLs elsewhere — this is the property bag doing exactly what it's supposed to do, and a fixed-column redesign would make this specific case *worse*, not better.

It is not rated "Fully Structured" because the *vocabulary* half of the pattern — the half `06_EquipmentDefinitions` already supplies for Equipment — doesn't exist for Technical. `TechnicalField` is free text with no canonical list, no canonical unit binding, and no way to express a measurement qualifier (mirror state, wheel size, MRO-vs-kerb) as anything other than a hand-typed suffix on the name itself. The four pieces of live-data evidence above (Kerb weight, Width, WLTP range, six orphaned rows) are the direct, observed consequence of that missing half — not hypothetical risks.

---

# Structural Options Considered

| Option | Recommendation | Rationale |
| --- | --- | --- |
| Fixed-column table (one column per technical characteristic) | **Reject** | Would require one column per property across all vehicles ever added; sparse, model-specific facts (frunk volume, trailer weight, wheel-size-dependent range) would force wide, mostly-NULL rows and require a schema change (new column) every time any single vehicle introduces a new measurable fact. Directly the opposite of what a 50-vehicle catalog needs. |
| Property bag exactly as-is (free-text `TechnicalField`, no catalog) | **Reject — status quo, already showing drift at 5 vehicles** | This is what's implemented today. The Kerb weight / Width / WLTP range naming variants above are the observed cost: nothing constrains what string a contributor types, so semantically-identical facts fragment into multiple labels that don't join. Cheap to fix by hand now; not cheap at 2,000–5,000 rows. |
| Property bag + canonical field catalog (mirrors `05_Equipment`/`06_EquipmentDefinitions`) | **Adopt** | Introduce a Reference worksheet (e.g. `13_TechnicalFieldDefinitions`) owning `TechnicalFieldID`, canonical `Name`, canonical `Unit`, `Description`, following exactly the pattern `06_EquipmentDefinitions` already established and already proved out in this same workbook. `Technical.TechnicalField` becomes a foreign key into that catalog instead of free text — the same relationship `Equipment.EquipmentDefinitionID` already has. A separate, optional `Qualifier` column on `Technical` itself (e.g. `Name="Width"`, `Qualifier="mirrors folded"`) gives option-dependent and measurement-condition variants (wheel size, mirror state, MRO-vs-kerb) a structured home instead of a name suffix, without inventing a new entity for it. |

---

# Risks

**Risk of adopting the catalog now:**

* Adds a new Reference worksheet and a schema change to an existing Operational worksheet (`04_Technical` gains a `Qualifier` column; `TechnicalField` becomes constrained rather than free text) — under `docs/CONTRIBUTING.md` this is a methodology/schema change requiring an ADR and, per P4's open versioning question, likely a Framework Version bump.
* Existing 145 rows need to be reconciled against the new catalog (deciding, e.g., whether "Kerb weight" and "Kerb weight (MRO)" become one canonical field or two deliberately distinct ones) — real migration work, not free.

**Risk of leaving `Technical` as an ungoverned property bag:**

* The exact drift already observed (Kerb weight, Width, WLTP range naming fragmentation) will keep recurring, once per new vehicle per contributor, with no mechanism to catch it — at 50 vehicles this becomes a real data-quality problem that surfaces as silently-wrong comparisons (e.g. comparing Škoda's ambiguous `Width` against Tesla's explicit `Width (mirrors folded)` as if they were the same measurement).
* The six orphaned, contract-violating rows already sitting in the live data since v1.0 show the framework has no way to catch a `Technical` row that doesn't reference anything at all — this will not self-correct, and more rows like it should be expected as volume grows.
* Cross-vehicle and cross-configuration querying/filtering by technical characteristic — explicitly named as a Version 2 goal (AI-assisted evidence collection, per ARCH-006's own framing) — degrades into a text-matching problem instead of a structured join, the same failure mode ARCH-006 flagged for `Trim` before `04_Technical` was populated.

---

# Recommendation

**Targeted architecture enhancement recommended — extend, do not redesign.**

1. Keep `04_Technical`'s current shape: EAV fact table, one row per (Vehicle-or-Configuration, field, value), `Confidence`/`SourceID`/`LastUpdated` per row. This part scales correctly and should not change.
2. Add a canonical field catalog (`13_TechnicalFieldDefinitions` or similar), structurally identical to `06_EquipmentDefinitions`: `TechnicalFieldID`, `Name`, canonical `Unit`, `Description`. `Technical.TechnicalField` (or a renamed `TechnicalFieldID`) becomes a reference into this catalog rather than free text — closing the same loop `05_Equipment`/`06_EquipmentDefinitions` already closed for equipment.
3. Add an optional `Qualifier` column to `04_Technical` to hold measurement-condition variants (wheel size, mirror state, regulatory measurement standard) as structured data instead of a parenthetical suffix on the property name. This directly resolves the Width and WLTP-range cases and gives the Kerb-weight/MRO distinction a place to live if the review decides it's a real distinction rather than terminology drift.
4. Resolve the documentation/implementation drift regardless of the above: reconcile `workbook-schema.md`'s `Property` vs. the workbook's `TechnicalField`, and document the `FrameworkVersion` column that already exists on the live sheet but isn't in the schema doc.
5. Clean up the six orphaned template rows (no Vehicle/Configuration reference, no Value, no Source) — low-risk, no governance needed, just delete them.

This should proceed as a follow-up ADR (schema addition + Framework Version increment, per `docs/CONTRIBUTING.md`), not an ad hoc workbook edit — consistent with how ARCH-006 handed its Configuration Status recommendation to a subsequent governance step rather than implementing inline.

---

# Definition of Done

* [x] Current `Technical` entity evaluated against `entity-model.md`, `workbook-schema.md`, `implementation-contract.md`, `relationships.md`, ADR-003, and the live Reference Workbook data (all 151 rows of `04_Technical`, all 5 vehicles).
* [x] Fixed-column vs. property-bag vs. property-bag-with-catalog assessed against observed data, not hypotheticals.
* [x] Concrete live-data evidence gathered for each flagged gap (Kerb weight/MRO, Width, WLTP range wheel variants, six orphaned rows, Property/TechnicalField doc drift).
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `workbook-schema.md`, `entity-model.md`, `implementation-contract.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: keep the EAV shape, add a `TechnicalFieldDefinitions` catalog + `Qualifier` column via a follow-up ADR.
