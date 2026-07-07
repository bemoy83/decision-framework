# ADR-012 — Source Enrichment

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.8 (Next)

**Supersedes:** None

**Related:**

* audits/ARCH-012-Source-Evidence-Enrichment-Evaluation.md
* docs/02_criteria-and-weighting.md
* framework/architecture/enumerations.md
* framework/architecture/workbook-schema.md
* ADR-011 (Notes Strategy Policy)

---

# Context

ARCH-012 evaluated whether `09_Sources` should gain `PublishedDate`, `RetrievedDate`, and `Language` metadata ahead of adding more vehicles. It found `PublishedDate` and `RetrievedDate` already exist — `RetrievedDate` fully populated (87/87), `PublishedDate` populated only for the Sources where a real publish date exists and was captured (14/87; correctly blank for continuously-updated manufacturer configurator pages, incompletely captured for dated professional-review articles). `Language` does not exist and the data shows a real, substantial split: 31 of 87 Sources are Norwegian-domain, 56 are not.

The audit also resolved two older P6 concerns without proposing new columns: the Evidence Policy's Tier A/B/C hierarchy is already substantially expressible via the existing `Type` enum, and the "direct fetch vs. search-result synthesis" caveat is already a legitimate `Source.Notes` use under ADR-011's Notes policy.

---

# Decision

`09_Sources` gains a `Language` column, backed by a new small enumeration:

```text
Language: NB, EN, OTHER
```

All 87 existing rows are backfilled from each Source's own URL/Publisher (`.no` domains and Norwegian-language publishers → `NB`; everything else observed today → `EN`; `OTHER` reserved for future non-Norwegian, non-English sources).

`docs/02_criteria-and-weighting.md`'s Evidence Policy gains a documented mapping from the existing `Type` enum to the Tier A/B/C hierarchy:

| Type | Tier |
| --- | --- |
| MANUFACTURER, CERTIFICATION, DATABASE | Tier A |
| PROFESSIONAL_REVIEW, VIDEO | Tier B |
| COMMUNITY, OWNER_EXPERIENCE | Tier C |
| GOVERNMENT | Tier A |
| OTHER | Assessed case by case; default Tier C unless documented otherwise in the Source's own Notes |

No `PublishedDate` backfill migration is performed. Completion is adopted as an ongoing practice: `PublishedDate` shall be captured whenever a new `PROFESSIONAL_REVIEW`-type Source is added, and backfilled opportunistically when an existing Source is next touched for another reason.

`09_Sources`'s `Referenced By` list in `workbook-schema.md` is corrected to include `14_HardRequirementResults`, which already references `SourceID` (ADR-009) but was not added to this list at the time.

---

# Rationale

`Language` is added because the data already demonstrates real, present-day need (31/56 split), not a hypothetical future one, and because the framework's own relationship principles prefer an explicit, stable field over inferring the same fact from a URL string each time it matters.

The `Type`→Tier mapping is documented rather than implemented as a new column because `Type` already carries the necessary distinction — introducing a second, overlapping classification would violate the same "never duplicate information owned by another worksheet" principle ADR-011 just reinforced for `Notes`.

`PublishedDate` is deliberately not force-migrated. The audit found the gap is real but bounded (~35–40 rows) and not urgent enough to justify a one-shot backfill; treating it as an ongoing capture habit matches how the framework already treats similar completion gaps (e.g. `M003` winter-range verification, tracked as genuinely Unknown and filled in opportunistically rather than blocked on).

---

# Consequences

## Enumerations

`framework/architecture/enumerations.md` gains a **Language** enumeration (`NB`/`EN`/`OTHER`).

## Workbook Schema

`framework/architecture/workbook-schema.md`'s `09_Sources` Columns table gains `Language`; `Referenced By` corrected to include `14_HardRequirementResults`.

## Scoring Model Documentation

`docs/02_criteria-and-weighting.md`'s Evidence Policy section gains the `Type`→Tier mapping table.

## Reference Workbook

* `09_Sources` gains `Language`, backfilled for all 87 existing rows.
* README gains `Enum_Language`; `WorkbookVersion` 1.6 → 1.7; `FrameworkVersion` 1.7 → 1.8 (a Schema-type bump per ADR-010's rule — no existing evaluation requires re-scoring as a result).
* `11_DecisionLog` gains an entry recording this change (`VersionBumpType = SCHEMA`).

---

# Alternatives Considered

## Add a dedicated `Tier` column instead of documenting a `Type`→Tier mapping

Rejected. `Type` already carries this distinction for all but a handful of `OTHER`-classified Sources; a parallel `Tier` column would duplicate information `Type` already owns, the exact anti-pattern ADR-011 was just written to prevent.

## Add a dedicated `FetchMethod` column (direct fetch vs. search-result synthesis)

Rejected. This is a per-instance implementation caveat, not a structural fact about the Source itself (the same Source could in principle be fetched directly next time) — `Source.Notes` already correctly carries it under the ADR-011 policy.

## Force-migrate all missing `PublishedDate` values now

Rejected as disproportionate to the actual risk. None of the affected Sources have caused a wrong conclusion so far (the one close call, Tesla's M003 investigation, was caught manually); ongoing capture going forward is lower-effort and equally effective without a large one-shot research task.

---

# Migration Strategy

`Language` is backfilled for all 87 existing Sources as part of this ADR's implementation, using each Source's own URL/Publisher — no new research performed, purely a classification of already-known information. `PublishedDate` completion is left as ongoing practice, not migrated.

---

# Impact

Affected documents:

* docs/02_criteria-and-weighting.md
* framework/architecture/enumerations.md
* framework/architecture/workbook-schema.md

Affected workbook:

* `09_Sources` (`Language` column added and backfilled)
* README (`Enum_Language`, `WorkbookVersion`, `FrameworkVersion`)
* `11_DecisionLog` (new entry)

No changes are required to:

* `08_Evidence`, `04_Technical`, `05_Equipment`, `14_HardRequirementResults` — no reference shape changes, only a new attribute on the Source they already reference.

---

# Guiding Principle

> **Enrich the entity that already owns the fact rather than adding a second place to record something it already knows how to say.**
