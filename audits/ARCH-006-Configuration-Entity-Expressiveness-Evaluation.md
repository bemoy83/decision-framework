# ARCH-006 — Configuration Entity Expressiveness Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-06
**Framework Version:** 1.1
**Related:** ADR-002, ADR-003, entity-model.md, relationships.md, workbook-schema.md, data-flow.md, implementation-contract.md, enumerations.md

---

# Executive Summary

The current `Configuration` entity is architecturally sound and does not require a schema redesign. Real usage during the Tesla Model 3 reference dataset population surfaced two concrete gaps, but both are gaps between *documentation and implementation*, not gaps in the underlying model. First, `enumerations.md` already defines a `Configuration Status` enumeration, but no workbook column exposes it on `03_Configurations`, so Configuration lifecycle state is currently unrepresentable. Second, drivetrain and battery-variant information (e.g. "Long Range firehjulsdrift") is being folded into the free-text `Trim` field because `04_Technical` — the entity that already owns this kind of measurable, configuration-specific fact per ADR-003 — remains entirely unpopulated. Neither observation justifies adding new columns to `Configuration` itself; the first is a case of finishing a wiring already promised by the enumeration contract, and the second is a case of using an existing mechanism rather than inventing a new one. Net recommendation: a minor, targeted enhancement (Configuration Status) rather than any redesign of Configuration's shape.

---

# Current Assessment

**Adequately Expressive.**

`Configuration` correctly owns identity-level and purchasing-level information — `ConfigurationID`, `VehicleID`, `Market`, `Trim`, `BasePrice`, `ConfiguredPrice`, `Notes` — and this is sufficient to identify *which purchasable product* is being evaluated, consistent with ADR-003's framing of Configuration as "the purchasable product." The boundary between Configuration (identity/commercial variant) and Technical (measurable fact) remains coherent on paper.

It is not rated "Fully Expressive" because the current reference dataset shows real friction at the boundary:

* `Configuration Status` is a documented enumeration (`framework/architecture/enumerations.md`) explicitly scoped to "Configuration entities only," yet no worksheet column implements it. `03_Configurations` therefore cannot state whether `TESLA_MODEL_3_PERFORMANCE` is currently orderable, upcoming, or discontinued — only `02_Vehicles.Status` exists, and Vehicle status does not imply Configuration status (a Vehicle can be `AVAILABLE` while one of its trims is discontinued).
* `Trim` is currently carrying more than a trim designation. The live data reads `"Bakhjulsdrift"`, `"Long Range firehjulsdrift"`, `"Long Range bakhjulsdrift"`, `"Performance"` — i.e. drivetrain (RWD/AWD) and battery variant (Standard/Long Range) are encoded inside a single free-text field, because `04_Technical` (the entity documented to own exactly this kind of configuration-specific measurable fact — see ADR-003's own example, "charging capability... wheel size... option-dependent specifications") has zero populated rows.

Both observations describe *incomplete use of existing architecture*, not a structural insufficiency in `Configuration`. This distinction matters: the fix is to finish wiring what is already documented, not to expand Configuration's ownership.

It is rated "Adequately" rather than "Partially" Expressive because, unlike a true architectural gap, both issues have a documented home already — `Configuration Status` in `enumerations.md`, drivetrain/battery facts in `04_Technical` — the home is simply unbuilt or unused.

---

# Candidate Attribute Review

| Attribute | Recommendation | Rationale |
| --------- | -------------- | --------- |
| Configuration Status | **Add to Configuration** | Describes the Configuration itself (purchasability lifecycle), is not measurable/Technical, cannot be inferred from `Vehicle.Status` (a Vehicle can be `AVAILABLE` while a specific trim is `DISCONTINUED`), and the enumeration is *already part of the framework contract* (`enumerations.md`) — this is closing a documented gap, not introducing a new concept. Its absence directly reduces explainability: the framework cannot currently state why an eligible-looking Configuration should or shouldn't appear in a live recommendation. |
| Drive Type | **Already represented elsewhere** | Drive type (RWD/AWD) is a measurable, configuration-specific characteristic — squarely inside `04_Technical`'s stated purpose, and analogous to ADR-003's own listed example of Configuration-level Technical data ("charging capability," "wheel size"). Adding a `Configuration.DriveType` column would duplicate what `Technical` is designed to hold once populated. The real problem is that `04_Technical` is currently an empty template — the fix is populating it, not extending `Configuration`. |
| Battery Variant | **Already represented elsewhere** | The numeric fact (battery capacity in kWh) already has a scaffolded `04_Technical` field (`Battery net`). A separate categorical "variant" label (e.g. "Long Range") would duplicate that value once populated and would violate the Ownership Validation rule in `workbook-schema.md` ("Information shall exist in only one worksheet"). Same underlying issue as Drive Type: an unpopulated Technical sheet, not a missing Configuration field. |
| Marketing Name | **Already represented elsewhere** | `Trim` is already functioning as the market-facing name in the live dataset (e.g. `"Long Range firehjulsdrift"`). A separate `MarketingName` field would duplicate `Trim` without a demonstrated distinct purpose. This becomes **Requires further discussion** only if the framework later wants to formally split `Trim` into a stable internal code vs. a market-facing display string — no evidence from the current dataset shows that split is needed yet. |

---

# Risks

**Risk of adding attributes now:**

* Extending `Configuration` to absorb Drive Type / Battery Variant would blur the Configuration/Technical boundary that ADR-003 and the entity model deliberately established, weakening the "entity owns only what it creates" principle for a problem that already has a designated owner (`Technical`).
* Any addition to `03_Configurations` requires touching a document marked `Status: Locked` (`workbook-schema.md`) and triggers the framework's versioning rule ("changes to... methodology... create a new Framework Version"), which is a real governance cost — it should only be spent on gaps that are actually structural.

**Risk of leaving the model unchanged (i.e. doing nothing at all, including not closing the Configuration Status gap):**

* Contributors will keep encoding drivetrain and battery variant into free-text `Trim`, as already observed in the Tesla Model 3 dataset. This undermines the framework's own explainability and minimal-duplication principles and will compound as more vehicles are added — cross-vehicle filtering or comparison by drivetrain becomes a text-parsing problem instead of a structured query, which directly works against the Version 2 goal of AI-assisted evidence collection.
* Without `Configuration Status`, the framework cannot distinguish a currently-purchasable trim from a discontinued one at the entity level that actually matters for a purchase decision (Configuration), which is a direct tension with the framework's stated mission of supporting *current* purchasing decisions, not just historically accurate ones.

---

# Recommendation

**Minor architecture enhancement recommended.**

Specifically, and narrowly:

1. Add a `Status` column to `03_Configurations` backed by the already-documented `Configuration Status` enumeration. This is not a new concept — it closes an existing gap between `enumerations.md` and `workbook-schema.md`/the workbook itself, following the same pattern already used for `02_Vehicles.Status`.
2. No new columns on `Configuration` for Drive Type, Battery Variant, or Marketing Name. The correct next action there is operational, not architectural: begin populating `04_Technical` with Configuration-scoped rows (e.g. `Property = "Drive Type"`, `Property = "Battery Variant"`) so `Trim` can revert to being a genuine trim designation instead of a catch-all string.

This should proceed through the framework's normal governance path (ADR + Framework Version increment for the schema addition), not as an ad hoc workbook edit, consistent with `docs/CONTRIBUTING.md`'s rule that weighting/schema changes require discussion, documentation, and a version increment.

---

# Definition of Done

* [x] Current Configuration entity evaluated against `entity-model.md`, `relationships.md`, `workbook-schema.md`, `data-flow.md`, `implementation-contract.md`, ADR-002, ADR-003, and the live Reference Workbook data.
* [x] Every candidate attribute (Configuration Status, Drive Type, Battery Variant, Marketing Name) assessed against the stated evaluation criteria.
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `workbook-schema.md`, `entity-model.md`, `implementation-contract.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: Minor architecture enhancement (Configuration Status only).
