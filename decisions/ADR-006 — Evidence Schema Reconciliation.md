# ADR-006 — Evidence Schema Reconciliation

**Status:** Accepted

**Date:** 2026-07-06

**Framework Version:** 1.3 (Next)

**Supersedes:** None

**Related:**

* ADR-003 — Configuration Is The Evaluated Entity
* framework/architecture/entity-model.md
* framework/architecture/relationships.md
* framework/architecture/workbook-schema.md
* framework/architecture/enumerations.md

---

# Context

A "thin vertical slice" exercise took one Configuration (`TESLA_MODEL_3_LONG_RANGE_RWD`) through the full Evidence → Review → Score pipeline using real, sourced data, to validate that the documented methodology actually works end to end.

Populating real Evidence records exposed a divergence between the documented `08_Evidence` worksheet schema and the schema actually implemented in the Reference Workbook.

Documented schema (`framework/architecture/workbook-schema.md`):

```text
EvidenceID, VehicleID, ConfigurationID, Observation, SourceID, Confidence
```

Actual workbook schema (verified against the live file):

```text
EvidenceID, VehicleID, ConfigurationID, ReviewID, Claim, Evidence, SourceID, Confidence
```

Two problems follow from this divergence.

First, the workbook exposes a `ReviewID` column on `08_Evidence`. `framework/architecture/relationships.md` states explicitly that "Evidence does not reference Reviews... This relationship intentionally preserves a one-way flow of information" (Review references Evidence, never the reverse). A populated `ReviewID` on an Evidence record would silently violate this invariant. The column exists but has never been populated, which is exactly the kind of latent structural risk this ADR should close rather than leave for someone else to discover the hard way.

Second, the workbook splits the documented single `Observation` field into two undocumented fields, `Claim` and `Evidence`. Neither has a documented purpose. In particular, "Claim" invites interpretation-flavoured text, which risks Evidence encroaching on Review's documented responsibility ("Reviews interpret Evidence... Evidence never contains interpretation").

Separately, `framework/architecture/relationships.md` documents Review → Evidence as a one-to-many relationship ("Every Review references one or more Evidence records"), but `07_Reviews` — in both the documentation and the live workbook — has only a single scalar `EvidenceID` column. During the vertical slice, a comma-separated list of `EvidenceID` values was used as a working convention to represent multiple references. This convention was never ratified.

---

# Decision

The `08_Evidence` worksheet shall conform to the documented schema:

```text
EvidenceID, VehicleID, ConfigurationID, Observation, SourceID, Confidence
```

The `ReviewID` column is removed. Evidence shall never reference a Review, in either direction other than the documented Review → Evidence reference.

The `Claim` and `Evidence` columns are merged into a single `Observation` field, formatted as:

```text
<topic>: <source-attributed observation>
```

This format keeps the "topic" fragment purely descriptive (what is being observed), never evaluative, so it cannot be mistaken for interpretation.

The `07_Reviews.EvidenceID` column shall hold one or more Evidence identifiers as a comma-separated list. This is the ratified representation of the documented Review → Evidence one-to-many relationship. A dedicated junction worksheet is explicitly rejected as unnecessary complexity at current and foreseeable cardinality (see Alternatives Considered).

The `Evidence Type` enumeration (`framework/architecture/enumerations.md`) remains defined but unused in the workbook. This ADR does not introduce an `EvidenceType` column. That gap pre-dates this ADR, is independent of the schema divergence being resolved here, and is deliberately deferred rather than silently dropped.

---

# Rationale

Documentation is the source of truth (ADR-001). Where implementation and documentation disagree, documentation governs until a decision changes it. This ADR either brings the workbook into line with the documented schema (the `Observation` merge, the `ReviewID` removal) or ratifies a workbook convention that documentation was silent on (the comma-separated `EvidenceID` list), so that the next contributor does not have to reverse-engineer intent from a one-off vertical-slice script.

Removing `ReviewID` from `08_Evidence` closes a structural risk before it is ever exercised, rather than after a real ReviewID gets written into it and the one-way flow is broken in live data.

---

# Consequences

## Entity Model / Relationships

No relationship changes. `relationships.md` already correctly documents Review → Evidence as 1:N and Evidence as not referencing Review; this ADR brings the workbook into compliance with that existing model rather than changing it.

## Workbook Schema

`framework/architecture/workbook-schema.md` is updated:

* `08_Evidence` Columns table: unchanged (already documents `Observation`); Notes gain a line clarifying the workbook previously diverged and has been reconciled, and that `EvidenceType` remains an explicitly deferred gap.
* `07_Reviews` Columns table: the `EvidenceID` description is changed from "Supporting evidence" to explicitly state it may hold one or more Evidence identifiers, comma-separated.
* Validation Rules gain a rule: every comma-separated token in `07_Reviews.EvidenceID` must resolve to an existing `EvidenceID`.

## Reference Workbook

`08_Evidence`: `ReviewID` column removed; `Claim` + `Evidence` merged into `Observation` for all existing records. `07_Reviews.EvidenceID` values already conform (already comma-separated where more than one Evidence record applies) and require no migration.

---

# Alternatives Considered

## Keep `Claim` and `Evidence` as two distinct, newly-documented fields

Rejected. This would enshrine a distinction the framework's Evidence/Review separation does not want — "Claim" is definitionally adjacent to interpretation, which is Review's job, not Evidence's.

## Introduce a dedicated Review↔Evidence junction worksheet

Rejected as premature normalization. Current and foreseeable cardinality (a handful of Evidence records per Review) does not justify a new worksheet and the added referential-integrity surface it would bring. A comma-separated list on the existing scalar column satisfies "Explicit Relationships... using stable identifiers" without new structure. This may be revisited if cardinality grows materially.

## Populate `EvidenceType` now, since the column gap was noticed

Rejected as out of scope. It is a separate, pre-existing gap unrelated to the schema divergence this ADR resolves. Bundling it here would mix an unrelated enhancement into a reconciliation ADR, which this project's contribution guidelines discourage.

---

# Migration Strategy

Framework Version 1.2 workbook data remains valid under the interpretation that `Claim`+`Evidence` together constituted the observation. Framework Version 1.3 requires the merged `Observation` field and the removal of `ReviewID`. The 10 Evidence records created during the vertical slice are migrated as part of this ADR's implementation; no data is discarded, only reformatted.

---

# Impact

Affected documents:

* framework/architecture/workbook-schema.md

Affected workbook:

* `08_Evidence` (column removal, column merge)
* `07_Reviews` (documentation clarification only; no data change required)

No changes are required to:

* entity-model.md
* relationships.md
* data-flow.md
* implementation-contract.md
* enumerations.md

---

# Guiding Principle

> **Evidence documents what is known, in one unambiguous field, traceable to one or more Sources. It is never the place where interpretation — or a reference back to the Review that interprets it — quietly creeps in.**
