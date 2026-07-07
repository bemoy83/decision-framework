# Vehicle Onboarding Checklist

**Purpose:** ordered steps for adding one new Vehicle (with at least one Configuration) to the Reference Workbook, end-to-end (ADR-013).

This checklist repeats short quick-reference copies of rules defined authoritatively elsewhere. Where it links to `framework/architecture/workbook-schema.md` or `docs/02_criteria-and-weighting.md`, that document is the source of truth — this checklist is a pointer, not a restatement (ADR-011's own discipline, applied here to a template instead of a Notes field).

---

# Before You Start

- [ ] Confirm the Vehicle is in scope: BEV, Norwegian market, ownership-horizon-relevant (see `docs/02_criteria-and-weighting.md` Hard Requirements).
- [ ] If the manufacturer's own naming is ambiguous between two distinct products (trims vs. genuinely separate model lines), resolve this with the user before entering any data — don't assume.
- [ ] Decide which Configuration will be the flagship (the one that receives full Evidence/Review/Score/OverallScore treatment) — see `workbook-schema.md`'s Flagship Configuration Scope section (ADR-013). Every other Configuration only needs Technical/Equipment/HardRequirementResults.

# Step 1 — Vehicle

- [ ] Add one `02_Vehicles` row. `VehicleID` is free-text, name-based (e.g. `BYD_ATTO3`).

# Step 2 — Configurations

- [ ] Add one `03_Configurations` row per real, purchasable trim. `ConfigurationID` follows `<VehicleID>_<Trim>`.
- [ ] Set `HardRequirementOverride = FALSE` unless a confirmed override applies (ADR-009).

# Step 3 — Technical

- [ ] Add `04_Technical` rows for every `13_TechnicalFieldDefinitions` field you can source, at Vehicle level if shared across Configurations, or Configuration level if it varies by trim.
- [ ] Default `Confidence = MEDIUM` unless multiple independent sources agree (`HIGH`) or it's a sibling/cross-Configuration inference (`LOW`) — see `workbook-schema.md`'s Confidence Defaults (ADR-013).
- [ ] Use `python3 scripts/onboarding_helpers.py next-id 04_Technical` for the next `TechnicalID`.

# Step 4 — Equipment

- [ ] Add `05_Equipment` rows for every relevant `06_EquipmentDefinitions` entry, per Configuration. At minimum, cover `EQ_ACC` and `EQ_REAR_CAMERA` (needed for Hard Requirements M004/M005 in Step 5).
- [ ] Same Confidence-default rule as Step 3.

# Step 5 — Hard Requirement Results

- [ ] Add one `14_HardRequirementResults` row per Configuration × HARD Criterion (M001–M005), for **every** Configuration, not just the flagship (ADR-009).
- [ ] Do this *before* writing any Review — catching a Hard Requirement failure late in the process has been a repeated, documented mistake in this framework's history (see `DEC_M002_VIOLATION`).
- [ ] Apply sibling-inference where a Configuration lacks its own fact but a sibling's fact answers the requirement — `LOW` confidence, blank `SourceID`, the inference stated in `Reason` (ADR-009).
- [ ] If any Hard Requirement comes back FAIL: stop and ask the user whether to exclude the Configuration from scoring entirely, or keep it per an explicit `HardRequirementOverride = TRUE`. Don't default to either silently.

# Step 6 — Sourcing (flagship Configuration only, from here on)

- [ ] Work the Source Checklist below, in order, before falling back to broader web search.
- [ ] **Every claim sourced via an AI web-search summary must be independently verified by direct retrieval of the actual source page before being cited.** If it cannot be confirmed, record Unknown — never a downgraded-confidence guess (`docs/02_criteria-and-weighting.md`'s Sourcing Verification rule, ADR-013).
- [ ] Add `09_Sources` rows for everything cited. `SourceID` is free-text (`SRC_<name>`).

## Source Checklist (Norwegian-market EVs), in order

1. The manufacturer's official `.no` configurator/spec page
2. NAF bilguiden
3. The elbil.no test archive
4. Motor.no
5. bil24.no
6. One English-language outlet (What Car / Cars.com / Autocar)
7. Only fall back to broader web search after 1–6, subject to the Sourcing Verification rule above

# Step 7 — Evidence

- [ ] Add `08_Evidence` rows for the flagship Configuration's documented observations, each with a `SourceID`.
- [ ] Where evidence was gathered on a different (sibling) trim than the flagship, say so explicitly in the Observation text and justify why it still applies (shared platform/component) — the established "tested on X, but shares Y with Z" citation pattern.

# Step 8 — Reviews

- [ ] Add `07_Reviews` rows for the flagship Configuration, one per qualitative Category, each referencing ≥1 `EvidenceID`, with a Score (1–5) and Confidence.
- [ ] Use `python3 scripts/onboarding_helpers.py next-id 07_Reviews` for the next `ReviewID`.

# Step 9 — Scoring

- [ ] Add `10_Scoring` rows (formulas, per ADR-005) for every `WEIGHTED`+`Active` Criterion with a supporting Review.
- [ ] Add one `12_OverallScores` row for the flagship Configuration.
- [ ] Sanity-check before committing: `python3 scripts/onboarding_helpers.py score <ConfigurationID>` — reproduces the expected `OverallScore`/`CoveragePercent` without needing LibreOffice to recalculate the live formulas.
- [ ] If `15_Dashboard` exists, add the new flagship Configuration's ID to its Decision Summary/Category Breakdown/Coverage Grid helper-ID columns, and extend the Category Breakdown chart's series range to include it.

# Step 10 — DecisionLog

- [ ] Add an `11_DecisionLog` row summarizing the onboarding. `VersionBumpType = NONE` unless a schema/methodology change was also made in the same session.
- [ ] Use `python3 scripts/onboarding_helpers.py next-id 11_DecisionLog --category DATA` (or the relevant category) for the next `DecisionID`.

---

# Quick Reference

## ID Prefix Table

See `framework/architecture/workbook-schema.md`'s Naming Conventions section (ADR-013) for the authoritative table. Summary: `TECH_`/`EQ_`/`REV_`/`EV_`/`SCORE_`/`OVSC_`/`HRR_` are 6-digit zero-padded sequential; `DEC_<CATEGORY>_` is 3-digit; `SRC_`/`02_Vehicles`/`03_Configurations` are free-text.

## Confidence Defaults

See `workbook-schema.md`'s Confidence Defaults section (ADR-013) for the authoritative rule. Summary: default `MEDIUM`; `HIGH` only for multi-source-confirmed or trivial facts; `LOW` for sibling/cross-Configuration inference.

---

# Guiding Principle

> Every vehicle should be onboardable by following this list alone, without re-deriving convention from memory or from the previous vehicle's data.
