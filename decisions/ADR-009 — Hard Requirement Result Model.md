# ADR-009 — Hard Requirement Result Model

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.6 (Next)

**Supersedes:** None

**Related:**

* ADR-004 — Configuration Lifecycle Status
* ADR-005 — Scoring Mechanics
* audits/ARCH-009-Hard-Requirement-Model-Evaluation.md
* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md
* framework/architecture/implementation-contract.md
* framework/architecture/relationships.md
* framework/architecture/enumerations.md
* docs/03_scoring-model.md

---

# Context

ARCH-009 evaluated whether Hard Requirement compliance — today expressed entirely through `03_Configurations.Notes` prose and `11_DecisionLog` cross-references — should become its own structured model. It found that every fact each Hard Requirement needs already exists in structured form elsewhere in the workbook (`04_Technical` for length and winter range; `05_Equipment` for adaptive cruise control and rearview camera), but nothing joins those facts into a queryable per-Configuration PASS/FAIL/UNKNOWN answer. Reconstructing that answer today requires reading every Configuration's Notes paragraph individually, and doing so by hand has already produced one confirmed miss (Tesla's M002 length violation, undetected for 8+ criteria) and several silent gaps (3 of 4 Tesla Configurations and 2 of 3 Volvo Configurations have no recorded adaptive-cruise-control or rearview-camera answer at all, with no explicit Unknown flag; Kia's Equipment data already answers both requirements but its Notes never says so).

The audit also found a specific reason the result cannot be a pure calculated formula (unlike `10_Scoring.RawScore`): a naive `≥300km` threshold check against Renault 4 Techno's only winter-range data point (287km, wrong trim, mild rather than winter conditions) computes a confident FAIL, while the framework's own existing judgment — correctly — is UNKNOWN with a flagged concerning signal. A formula cannot represent "the number crossed the line, but the evidence behind it is weak"; a contributor-authored conclusion, informed by but not mechanically dictated by the underlying fact, can.

Separately, the audit surfaced but did not resolve how a **confirmed FAIL kept for comparison by explicit user override** (Tesla's four Configurations, kept scored despite the M002 violation per the user's standing instruction) should be represented, since today this is prose in `03_Configurations.Notes` and a hand-typed static string in `12_OverallScores.Notes`.

---

# Decision

A new Operational worksheet, `14_HardRequirementResults`, is introduced — structurally closer to `07_Reviews` (a contributor-authored conclusion) than to `10_Scoring` (a pure formula):

```text
HardRequirementResultID, ConfigurationID, CriterionID, Result, Confidence, Reason, SourceID, FrameworkVersion
```

* `CriterionID` references a `01_Criteria` row where `Type = HARD` — no new Criterion concept is introduced; this is the same distinction the framework already makes.
* `Result` is a new enumeration: `PASS`, `FAIL`, `UNKNOWN`.
* `Confidence` reuses the existing `Enum_Confidence` enumeration (`HIGH`/`MEDIUM`/`LOW`/`UNKNOWN`), the same field already used by Technical, Equipment, Evidence, and Review.
* `Reason` is free text: a contributor's interpretation of the supporting fact, following the same principle ADR-008 already established for Review — it should cite the underlying Technical/Equipment fact and, where the evidence is weak, say so explicitly (as in the Renault 4 Techno case), rather than only restating a threshold comparison.
* `SourceID` references `09_Sources` directly, not `08_Evidence`. This deviates from ARCH-009's initially proposed shape (which suggested `EvidenceID`, mirroring Review). On implementation, Hard Requirement facts are found to draw directly from `04_Technical`/`05_Equipment` records, which are themselves already Source-attributed — requiring a Result to also reference an Evidence record would mean minting Evidence rows solely to satisfy a schema constraint, duplicating sourcing information that already exists one level down. Where a Result is inferred from a sibling Configuration's fact rather than its own Source (see below), `SourceID` is left blank and the inference is stated in `Reason` instead.

One row exists per Configuration × HARD-type Criterion. All 18 live Configurations are backfilled against all 5 Hard Requirement criteria (M001–M005), 90 rows total, using the same judgment ARCH-009 already demonstrated is necessary:

* Where a Configuration's own Technical/Equipment fact answers the requirement, `Result`/`Confidence`/`SourceID` are taken directly from it.
* Where a Configuration has no fact of its own but a sibling Configuration of the same Vehicle does, and the workbook's existing Notes already reasoned from that sibling (Volvo P5/P5 Long Range's adaptive cruise control, inferred from the confirmed P8 AWD), the inference is preserved as `PASS` at `LOW` confidence rather than discarded to `UNKNOWN` — consistent with the brand/sibling-fallback principle already established for Long-Term Ownership criteria (lower confidence, not automatically a worse or blocked result).
* Where the existing Notes deliberately left a fact unresolved rather than inferring it (Volvo's rearview camera, explicitly "left Unknown rather than assumed"), that choice is preserved as `UNKNOWN`, not silently upgraded.
* Where no fact and no reasoned inference exist at all (Tesla `RWD`/`LONG_RANGE_AWD`/`PERFORMANCE`'s adaptive cruise control and rearview camera), the result is `UNKNOWN` at `LOW` confidence with a `Reason` flagging the gap explicitly, rather than left silent as today.
* Winter range (M003) is `UNKNOWN` for every Configuration except Volvo `P8_AWD` (`PASS`, matching trim, independently tested) — Renault 4 Techno is recorded as `UNKNOWN` with the mismatched-trim/mild-conditions caveat in `Reason`, not `FAIL`.

`03_Configurations` gains a new column, `HardRequirementOverride` (Boolean), answering: is this Configuration deliberately kept in scoring/comparison despite a confirmed Hard Requirement FAIL, by explicit user decision? Set `TRUE` for all four Tesla Configurations (the only case where this currently applies) and `FALSE` for all others, including Renault's `EVOLUTION_URBAN`/`EVOLUTION_COMFORT` — which fail M004 and remain correctly excluded from `10_Scoring`, the default (non-overridden) behavior. This is added to `Configuration` rather than to `HardRequirementResult` because it is a decision about the Configuration's overall eligibility for comparison, not a property of any single requirement's result.

---

# Rationale

`docs/03_scoring-model.md` already describes Hard Requirements as Stage 1 of the pipeline — evaluated before Technical Facts, Evidence, or Review. This ADR does not change that intent; it gives it a structure that can actually be checked and queried, the same way ADR-005 gave the already-intended Overall Score concept a real, reproducible implementation.

Modeling `Result` like `Review` rather than like `Score` follows directly from the Renault 4 Techno evidence: Hard Requirement facts vary in evidentiary quality (mismatched trims, proxy conditions, sibling-Configuration inference) in exactly the way Review already exists to capture for qualitative criteria. A pure formula would either lose this nuance or require per-row formula exceptions, which is worse than a documented, contributor-authored conclusion.

`HardRequirementOverride` living on `Configuration` rather than on `HardRequirementResult` follows ADR-004's precedent: Configuration already owns its own eligibility/lifecycle state independent of any single measured fact, and the override is fundamentally the same kind of decision — a Configuration-level choice about whether to proceed, not a fact about the world.

---

# Consequences

## Entity Model

`entity-model.md` gains a **HardRequirementResult** entity: owns `result`, `confidence`, `reason`; references `Configuration`, `Criterion`, and optionally `Source`. `Configuration`'s `Owns` list gains `hard requirement override`.

## Relationships

`relationships.md` gains `Configuration → HardRequirementResult` (1:N) and `HardRequirementResult → Criterion` (N:1, restricted to `Type = HARD`).

## Workbook Schema

`framework/architecture/workbook-schema.md` gains a `14_HardRequirementResults` worksheet specification (Operational, same governance as `07_Reviews` — contributor-authored, not auto-calculated). `03_Configurations`' Columns table gains `HardRequirementOverride`. Validation Rules gain a rule requiring every `HardRequirementResult` to reference an existing Configuration and a Criterion where `Type = HARD`.

## Enumerations

`enumerations.md` gains a **Hard Requirement Result** enumeration: `PASS`, `FAIL`, `UNKNOWN`.

## Scoring Model Documentation

`docs/03_scoring-model.md` Stage 1 gains a line pointing to `14_HardRequirementResults` as the concrete implementation of the Hard Requirement gate already described there.

## Reference Workbook

* New sheet `14_HardRequirementResults`: 90 rows (18 Configurations × 5 Hard Requirement criteria M001–M005), backfilled per the rules above.
* `03_Configurations`: new `HardRequirementOverride` column; `TRUE` for the 4 Tesla Configurations, `FALSE` for the other 14.
* README: new `Enum_HardRequirementResult` column (`PASS`/`FAIL`/`UNKNOWN`); `WorkbookVersion` 1.4 → 1.5; `FrameworkVersion` 1.5 → 1.6; `LastUpdated` set to 2026-07-07.
* `11_DecisionLog` gains an entry recording this migration.

---

# Alternatives Considered

## `Result` as a calculated formula over `04_Technical`/`05_Equipment` thresholds

Rejected, per ARCH-009's Renault 4 Techno evidence — a naive threshold comparison produces a confidently wrong answer where the correct answer is `UNKNOWN` with a caveat. Rejected for the same reason `10_Scoring.Explanation`'s free text (not a formula) exists alongside the calculated `RawScore`/`WeightedScore` — some information here is judgment, not arithmetic.

## `HardRequirementOverride` as a new `Configuration Status` enum value

Rejected. `Configuration Status` (ADR-004) is deliberately a commercial-lifecycle concept (`AVAILABLE`/`UPCOMING`/`DISCONTINUED`/`UNKNOWN`), independent of framework evaluation eligibility — Tesla's Configurations are genuinely commercially `AVAILABLE` (purchasable today) while being evaluation-ineligible per M002. Folding the override into the same enum would conflate two orthogonal concepts ADR-004 itself took care to keep separate.

## `Result` referencing `EvidenceID` instead of `SourceID`

Rejected on implementation, reversing ARCH-009's initial suggestion. Every Hard Requirement fact found during backfill traces directly to a `04_Technical` or `05_Equipment` record, which already carries its own `SourceID`. Requiring a separate Evidence record would mean creating Evidence rows purely to satisfy a schema constraint, not because a genuine documented observation exists beyond the Technical/Equipment fact itself.

## Rewiring `12_OverallScores`' hand-typed Tesla Notes formula to derive from `HardRequirementResult`/`HardRequirementOverride` automatically

Deferred, not rejected. This is a real, valuable follow-on (making the "NOT ELIGIBLE, kept for comparison" flag self-maintaining instead of a hand-typed string), but it touches `12_OverallScores`' formula logic, which is governed by ADR-005 and used across all 18 rows — a change of that shape deserves its own careful pass rather than being bundled into the Result model's introduction.

---

# Migration Strategy

Framework Version 1.5 workbook data remains valid under the interpretation that Hard Requirement compliance was recorded only in Notes prose. Framework Version 1.6 requires every Configuration to have one `HardRequirementResult` row per HARD-type Criterion. All 90 rows are backfilled as part of this ADR's implementation directly from existing `04_Technical`/`05_Equipment` facts and the judgment already recorded in `03_Configurations.Notes` and `11_DecisionLog` — no new research was performed; this is a structuring of already-known information, not new evaluation.

---

# Impact

Affected documents:

* framework/architecture/entity-model.md
* framework/architecture/workbook-schema.md
* framework/architecture/implementation-contract.md
* framework/architecture/relationships.md
* framework/architecture/enumerations.md
* docs/03_scoring-model.md

Affected workbook:

* `14_HardRequirementResults` (new sheet)
* `03_Configurations` (`HardRequirementOverride` column added)
* README (`Enum_HardRequirementResult`, `WorkbookVersion`, `FrameworkVersion`, `LastUpdated`)
* `11_DecisionLog` (new entry)

No changes are required to:

* `10_Scoring` / `12_OverallScores` (formula rewiring explicitly deferred, see Alternatives Considered)
* `docs/02_criteria-and-weighting.md` (its Hard Requirements table lists some concepts — Ownership Horizon, Primary Vehicle, Market — not yet implemented as `01_Criteria` rows; reconciling that list against the five implemented M001–M005 criteria is a separate, unscoped question this ADR does not address)

---

# Guiding Principle

> **A gate that only opens when someone happens to notice it isn't a gate. Every Configuration answers every Hard Requirement, explicitly — PASS, FAIL, or a documented UNKNOWN — before anyone asks whether it can be compared at all.**
