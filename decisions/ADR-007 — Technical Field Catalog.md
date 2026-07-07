# ADR-007 — Technical Field Catalog

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.5 (Next)

**Supersedes:** None

**Related:**

* ADR-003 — Configuration Is The Evaluated Entity
* ADR-006 — Evidence Schema Reconciliation
* audits/ARCH-006-Configuration-Entity-Expressiveness-Evaluation.md
* audits/ARCH-007-Technical-Entity-Property-Bag-Evaluation.md
* framework/architecture/entity-model.md
* framework/architecture/relationships.md
* framework/architecture/workbook-schema.md
* framework/architecture/implementation-contract.md

---

# Context

ARCH-007 evaluated whether `04_Technical` should be a fixed-column table or a property bag, ahead of scaling the framework from 5 vehicles / ~150 Technical rows toward a target of ~50 vehicles / 2,000–5,000 rows. It concluded the property-bag (entity-attribute-value) shape is correct for this domain — vehicles carry a long tail of sparse, model-specific facts (frunk volume, trailer weight, wheel-size-dependent range) that a fixed-column table would force into ever-growing, mostly-NULL columns.

The audit found the property bag is nevertheless incomplete: `05_Equipment` pairs its fact table with `06_EquipmentDefinitions`, a Reference worksheet owning the canonical vocabulary of what a feature is, so `Equipment.EquipmentDefinitionID` is a foreign key rather than free text. `04_Technical` has no equivalent. Its property-name column (`TechnicalField` in the live workbook; documented as `Property` in `workbook-schema.md`, itself a drift this ADR resolves) is raw, unvalidated free text.

Live-data evidence gathered during ARCH-007 (5 vehicles, 151 rows) showed this already producing drift at small scale:

* `Kerb weight` (Renault, Volvo — 7 rows) vs. `Kerb weight (MRO)` (Tesla — 4 rows): the two never co-occur on the same vehicle, and nothing distinguishes whether this is a deliberate methodological split (EU "mass in running order" vs. plain kerb weight are, in fact, different regulatory measures) or accidental terminology drift.
* `Width` (Škoda, Volvo — unqualified, mirror state unknown) vs. `Width (mirrors folded)` / `Width (mirrors unfolded)` (Tesla — explicit): Width is not safely comparable across manufacturers today.
* `WLTP range` vs. `WLTP range (18" wheels)` / `WLTP range (19" wheels)`: a single Tesla Configuration (`TESLA_MODEL_3_LONG_RANGE_RWD`) records a wheel-size-dependent option split by suffixing the property name, the only mechanism available today for the "option-dependent specifications" case ADR-003 itself anticipates.
* Six orphaned rows (`FrameworkVersion 1.0`) with neither `VehicleID` nor `ConfigurationID` populated, no `Value`, no `SourceID` — a direct violation of `implementation-contract.md`'s mandatory rule that Technical references either one Vehicle or one Configuration, never caught because nothing validates it.

None of this is damaging at 151 rows. It is the mechanism by which drift will compound, silently and per-contributor, at 2,000–5,000 rows.

---

# Decision

`04_Technical`'s fact-table shape is retained unchanged: one row per (Vehicle-or-Configuration, field, value), with `Confidence`, `SourceID`, and `LastUpdated` per row.

A new Reference worksheet, `13_TechnicalFieldDefinitions`, is introduced — structurally the same pattern as `06_EquipmentDefinitions`:

```text
TechnicalFieldID, Name, Unit, Description, Notes
```

`04_Technical.TechnicalField` is repurposed from a free-text display string to a foreign key referencing `13_TechnicalFieldDefinitions.TechnicalFieldID`, the same relationship `Equipment.EquipmentDefinitionID` already has to `EquipmentDefinition`.

A new optional column, `Qualifier`, is added to `04_Technical` to hold measurement-condition variants — wheel size, mirror state, measurement standard — as structured data rather than a parenthetical suffix on the property name:

```text
TechnicalID, VehicleID, ConfigurationID, TechnicalField, Value, Unit, SourceID, Confidence, LastUpdated, FrameworkVersion, Qualifier
```

`Qualifier` is appended after the existing columns rather than inserted next to `TechnicalField`. The live workbook's `Unit` and `Confidence` columns carry column-anchored list validations (`Enum_Unit`, `Enum_Confidence`); inserting a column ahead of them would silently misalign those validations to the wrong column. Appending avoids that risk entirely — the relationship `Qualifier` describes (a modifier on `TechnicalField`) is expressed through documentation and the entity model, not through physical column adjacency.

The `FrameworkVersion` column already present on the live `04_Technical` sheet — undocumented in `workbook-schema.md` until now — is formally documented, following the same "every evaluation record stamps its FrameworkVersion" convention already used by `07_Reviews`, `10_Scoring`, and `12_OverallScores`.

Seventeen canonical fields are introduced, collapsing the 22 raw free-text strings observed in the live data:

| TechnicalFieldID | Name | Unit | Qualifier used for |
| --- | --- | --- | --- |
| TF_LENGTH | Length | MM | — |
| TF_WIDTH | Width | MM | Mirror state (folded / unfolded), where known |
| TF_HEIGHT | Height | MM | — |
| TF_GROUND_CLEARANCE | Ground clearance | MM | — |
| TF_KERB_WEIGHT | Kerb weight | KG | Measurement standard (e.g. "Mass in running order (MRO)"), where known |
| TF_MAX_TRAILER_WEIGHT | Max trailer weight | KG | — |
| TF_BOOT_VOLUME | Boot volume | L | — |
| TF_FRUNK_VOLUME | Frunk volume | L | — |
| TF_BATTERY_NET | Battery net | KWH | — |
| TF_AC_MAX_CHARGING | AC max charging | KW | — |
| TF_DC_MAX_CHARGING | DC max charging | KW | — |
| TF_WLTP_RANGE | WLTP range | KM | Wheel size, where the manufacturer publishes more than one figure |
| TF_WLTP_CONSUMPTION | WLTP consumption | KWH_100KM | — |
| TF_WINTER_HIGHWAY_RANGE_ESTIMATE | Winter highway range estimate | KM | — |
| TF_MOTOR_POWER | Motor power | KW | — |
| TF_TORQUE | Torque | NM | — |
| TF_TOP_SPEED | Top speed | KMH | — |

`Kerb weight` and `Kerb weight (MRO)` are **not** split into two canonical fields. EU "mass in running order" and plain kerb weight are legitimately different regulatory measures (MRO adds a standardized driver-mass allowance), but the live data gives no evidence contributors have been applying that distinction deliberately — Tesla's four configurations use the MRO label exclusively, Renault's and Volvo's seven use the plain label exclusively, which is equally consistent with careless terminology copying from each manufacturer's own spec sheet. Collapsing to one canonical field with an explicit, documented `Qualifier` makes the distinction visible and correctable per-row without asserting an answer this ADR has no evidence to support.

The six orphaned rows (no Vehicle/Configuration reference, no Value, no Source, `FrameworkVersion 1.0`) are deleted. They predate any real data and satisfy no referential-integrity rule.

---

# Rationale

Documentation is the source of truth (ADR-001), but where the workbook's existing convention is sound and the documentation is merely silent or stale, the workbook's convention is adopted into the documentation rather than churned to match a stricter historical text — the same call ADR-006 made for `07_Reviews.EvidenceID`. `TechnicalField` (workbook) is kept over `Property` (doc) on that basis: it is the clearer name and renaming 145 live values for a cosmetic sync would be pure churn. The undocumented `FrameworkVersion` column is documented as-is, since it already matches the framework-wide "every evaluation stamps its version" convention used elsewhere.

The `TechnicalFieldDefinitions` catalog is a direct application of a pattern the framework has already validated once (`Equipment`/`EquipmentDefinitions`) to a second entity showing the identical symptom (free-text names fragmenting the same fact into multiple unjoinable strings). It closes the gap while changing nothing about the parts of `Technical` that already work — the Vehicle-XOR-Configuration reference, the per-row Confidence/Source/LastUpdated provenance.

The `Qualifier` column resolves the "option-dependent specification" case ADR-003 already names as in-scope for Technical (wheel size, drivetrain-adjacent options) without introducing a new entity or a trim-tier grouping mechanism — deliberately the smaller change, consistent with ARCH-007's recommendation to extend rather than redesign.

---

# Consequences

## Entity Model / Relationships

A new entity, `TechnicalFieldDefinition`, is added to `entity-model.md`, structurally parallel to `EquipmentDefinition`. `Technical` gains a documented reference to `TechnicalFieldDefinition` in addition to its existing Vehicle-or-Configuration reference. `relationships.md` gains a `Technical → TechnicalFieldDefinition` many-to-one relationship, parallel to the existing `Equipment → EquipmentDefinition` section.

## Workbook Schema

`framework/architecture/workbook-schema.md` is updated:

* `04_Technical` Columns table: `Property` renamed to `TechnicalField` (now documented as a foreign key, not free text); `Qualifier` and `FrameworkVersion` columns added; References section gains `TechnicalFieldID`.
* A new `13_TechnicalFieldDefinitions` section is added (Purpose / Worksheet Type / Primary Key / References / Referenced By / Columns / Notes), mirroring `06_EquipmentDefinitions`.
* The worksheet overview table and the Reference Worksheets examples list both gain `13_TechnicalFieldDefinitions`.

## Reference Workbook

* New sheet `13_TechnicalFieldDefinitions`: 17 canonical field definitions, per the table above.
* `04_Technical`: `Qualifier` column added; all 145 live rows' `TechnicalField` values migrated from free-text display strings to `TechnicalFieldID` codes, with `Qualifier` populated for the five known variant cases (`Kerb weight (MRO)`, `Width (mirrors folded)`, `Width (mirrors unfolded)`, `WLTP range (18" wheels)`, `WLTP range (19" wheels)`). Each row's existing `FrameworkVersion` value is left untouched — this is a re-encoding of existing facts, not a re-verification of them.
* Six orphaned rows deleted.
* README worksheet: `WorkbookVersion` 1.3 → 1.4, `FrameworkVersion` 1.4 → 1.5, `LastUpdated` set to 2026-07-07.
* `11_DecisionLog` gains an entry recording this migration.

---

# Alternatives Considered

## Leave `TechnicalField` as free text, document a naming convention in prose only

Rejected. `workbook-schema.md` already documents a Vehicle-XOR-Configuration prose rule for Technical, and it is not enforced anywhere — a prose naming convention for `TechnicalField` would fail the same way. The six orphaned rows are direct evidence that undocumented-but-hoped-for conventions do not hold up; a referenceable catalog is the only mechanism ARCH-007 found that actually gets checked.

## Split `Kerb weight` and `Kerb weight (MRO)` into two distinct canonical fields now

Rejected. The live data does not show evidence of a deliberate, applied distinction — only that different manufacturers' source documents happened to use different labels. Asserting two canonical fields would encode a distinction the framework cannot currently back with sourced reasoning. The `Qualifier` column leaves room to reintroduce this split later if a future contributor sources evidence that a given vehicle's spec sheet deliberately reports MRO rather than kerb weight.

## Introduce a trim-tier/family grouping entity to handle option-dependent variance (the Renault boot-volume case)

Rejected for this ADR, per ARCH-007. The Tesla wheel-size case and the Renault trim-tier case are different shapes of variance (within one Configuration's sub-options vs. across sibling Configurations of one Vehicle); `Qualifier` solves the former cleanly. The latter is already representable today via per-Configuration `Technical` rows (with acceptable, documented duplication) and does not need a new entity to be unblocked. Revisit only if duplication volume becomes a real maintenance burden as more vehicles are added.

## Rename `TechnicalField` back to `Property` to match the historical documentation

Rejected. `TechnicalField` is the value the live workbook has used since early population and is the clearer of the two names; migrating 145 rows' column header for a purely cosmetic doc-sync would be churn with no benefit, and ADR-006 already established the precedent of updating documentation to match a sound workbook convention rather than the reverse.

---

# Migration Strategy

Framework Version 1.4 workbook data remains valid under the interpretation that `TechnicalField` held a free-text display name. Framework Version 1.5 requires `TechnicalField` to hold a `TechnicalFieldID` resolvable against `13_TechnicalFieldDefinitions`, and recognizes `Qualifier` as the structured home for measurement-condition variants. All 145 existing Technical rows are migrated as part of this ADR's implementation — reformatted only, no measured value is altered or discarded. The six orphaned, contract-violating rows are removed as part of the same migration.

---

# Impact

Affected documents:

* framework/architecture/workbook-schema.md
* framework/architecture/entity-model.md
* framework/architecture/relationships.md
* framework/architecture/implementation-contract.md

Affected workbook:

* `04_Technical` (column addition, value migration)
* `13_TechnicalFieldDefinitions` (new sheet)
* README (`WorkbookVersion`, `FrameworkVersion`, `LastUpdated`)
* `11_DecisionLog` (new entry)

No changes are required to:

* enumerations.md
* data-flow.md
* docs/02_criteria-and-weighting.md
* docs/03_scoring-model.md

---

# Guiding Principle

> **A property bag is only as reliable as the vocabulary behind it. Technical records what is measured; the Technical Field Catalog records what that measurement means — so the same fact, measured twice by two different contributors, resolves to the same row instead of two unjoinable strings.**
