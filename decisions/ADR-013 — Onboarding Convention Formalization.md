# ADR-013 — Onboarding Convention Formalization

**Status:** Accepted

**Date:** 2026-07-07

**Framework Version:** 1.8 (unchanged)

**Supersedes:** None

**Related:**

* ADR-009 (Hard Requirement Result Model)
* ADR-011 (Notes Strategy Policy)
* ADR-012 (Source Enrichment)
* docs/02_criteria-and-weighting.md
* framework/architecture/workbook-schema.md
* framework/templates/vehicle-onboarding-checklist.md
* scripts/onboarding_helpers.py

---

# Context

Onboarding BYD Atto 3 as the sixth vehicle surfaced five conventions that the last several onboarding sessions had followed consistently, but that existed only as an emergent pattern in the live workbook data — never written down. This directly contradicts `docs/CONTRIBUTING.md`'s own stated philosophy: "Documentation defines the framework... Data never defines methodology."

**Confidence defaults are undocumented.** `04_Technical` is 156/159 (98%) `MEDIUM`; `05_Equipment` is 58/68 (85%) `MEDIUM`; `07_Reviews` is 86/120 (72%) `MEDIUM`; `14_HardRequirementResults` is 49/100 (49%) `MEDIUM`, with 29 `LOW` and 22 `HIGH` — a deliberately wider spread, consistent with ADR-009's own judgment-based model for that worksheet specifically. `docs/02_criteria-and-weighting.md` and `docs/03_scoring-model.md` both define what `HIGH`/`MEDIUM`/`LOW` *mean*; neither states which to default to when recording a new fact.

**The ID-prefix convention exists only as one worked example per identifier type.** `framework/architecture/workbook-schema.md`'s Naming Conventions section shows one example block per identifier (`TECH_000001`, `EQ_000001`, etc.) but never states the rule those examples all follow, nor which worksheets are exceptions. One example has already drifted from reality: `ConfigurationID` is shown as `CONF_KIA_EV2_EXCLUSIVE_NO`, but every live `ConfigurationID` (including the real `KIA_EV2_FWD_LONG_RANGE_GT_LINE`) follows `<VehicleID>_<Trim>` with no `CONF_` prefix at all.

**"Exactly one flagship Configuration per Vehicle receives Evidence/Review/Score/OverallScore" is undocumented.** Confirmed empirically across all six Vehicles onboarded to date — Tesla (4 Configurations), Renault (6), Skoda (4), Volvo (3), Kia (1), BYD (2) — each has exactly one row in `12_OverallScores`. This is a distinct concern from ADR-009's sibling-Configuration inference rule: ADR-009 governs how a sibling's Technical/Equipment fact may be borrowed to infer a `HardRequirementResult`, once it is already settled which Configuration is being evaluated at all. Nothing in the documentation today says only one Configuration per Vehicle should receive the full evaluation pipeline — a contributor reading the docs alone could reasonably conclude every Configuration should be independently scored, which is not current practice and was never a deliberate one-time decision, only consistent emergent behaviour.

**No rule requires verifying an AI-search-summary claim against its cited source before citing it.** While onboarding BYD Atto 3, a web-search summary confidently asserted the car achieved "311 km" in a NAF winter-range test. Direct retrieval of NAF's own page for this vehicle showed only summer-2023 test data — no winter figure at all. The claim could not be confirmed and was correctly recorded as Unknown rather than assumed, but only because it happened to be double-checked, not because any documented process step required it.

**No standard source checklist, and no committed onboarding checklist, exist.** Each of the six vehicles onboarded so far followed a different, ad-hoc research path. `framework/templates/` holds `pull-request-template.md`, `architecture-review-template.md`, and `codex-issue-template.md` (an unrelated implementation-task template) — nothing specific to "add one new Vehicle end-to-end." The checklist that has guided each onboarding session existed only in the assisting AI's own private working notes, never as a repository artifact.

---

# Decision

**1. Confidence Defaults.** `framework/architecture/workbook-schema.md` gains a stated default: absent a specific reason to record a different level, new `04_Technical`, `05_Equipment`, `08_Evidence`, and `07_Reviews` records default to `MEDIUM` — a single credible source, not independently cross-verified. `HIGH` is reserved for facts confirmed by multiple independent sources, or facts that are definitionally trivial to verify (e.g. "is this Vehicle a BEV"). `LOW` is reserved for facts inferred from a sibling Configuration, a mismatched trim/condition proxy, or otherwise weak supporting evidence. `UNKNOWN` remains governed entirely by `docs/01_project-philosophy.md`'s existing "Unknown is better than assumed" principle — this default does not change when Unknown is the right answer, only what to record once a fact is actually recorded. `14_HardRequirementResults` is explicitly exempted from the `MEDIUM` default: ADR-009 already establishes that this worksheet spans the full range by design, and forcing a default there would contradict ADR-009's own judgment-based model.

**2. ID-Prefix Convention.** `workbook-schema.md`'s Naming Conventions section gains a stated rule ahead of its existing examples: every numeric-sequence identifier follows `<PREFIX>_<N-digit zero-padded sequence>`, one independent counter per worksheet, assigned in strictly increasing order with no reuse of a retired number. The prefix and digit-width are fixed per worksheet and never change. A full table is added covering all worksheets, including the free-text exceptions (`09_Sources`, `02_Vehicles`, `03_Configurations`), the category-tagged exception (`11_DecisionLog`'s `DEC_<CATEGORY>_<NNN>`, where `<CATEGORY>` is free text, not a closed enumeration), and the descriptive-suffix identifiers (`06_EquipmentDefinitions`'s `EQDEF_`, `13_TechnicalFieldDefinitions`'s `TF_`). The stale `ConfigurationID` example (`CONF_KIA_EV2_EXCLUSIVE_NO`) is corrected to match a real, current identifier — a documentation-to-match-reality fix in the same spirit as ADR-006/ADR-007, not a data change.

**3. Flagship Configuration Scope.** `workbook-schema.md` gains an explicit statement, placed immediately after the `12_OverallScores` specification: not every Configuration belonging to a Vehicle receives Evidence, Review, Score, and OverallScore records. By established practice, exactly one Configuration per Vehicle — the "flagship," typically the trim under serious purchase consideration — receives the full Evidence → Review → Score → OverallScore treatment. Sibling Configurations of the same Vehicle receive only `04_Technical` and `05_Equipment` records (and, per ADR-009, may have `14_HardRequirementResults` inferred from the flagship or another sibling). This is an evaluation-scope decision, distinct from and complementary to ADR-009's sibling-inference mechanism. Promoting a sibling Configuration to flagship status at any later time is always permitted and does not require a new ADR — it is a data-completeness action, not a methodology change.

**4. Sourcing Verification.** `docs/02_criteria-and-weighting.md`'s Evidence Policy gains a new rule: an AI-generated web-search summary may be used to *locate* a candidate source, but never to *confirm* what that source says. A claim shall not be cited as Evidence, or used to support a Review, until its existence and content have been independently verified by direct retrieval of the cited source's own page. Where direct retrieval cannot confirm the claim — the page does not contain the asserted figure, or the page is inaccessible — the fact shall be recorded as Unknown per `docs/01_project-philosophy.md`'s "Unknown is better than assumed" principle, not asserted at a merely reduced confidence. The BYD Atto 3 / NAF "311 km" case above is the motivating example.

**5. Standard Source Checklist.** A fixed, ordered checklist for Norwegian-market EV research is adopted for the flagship Configuration's sourcing: (1) the manufacturer's official `.no` page, (2) NAF bilguiden, (3) the elbil.no test archive, (4) Motor.no, (5) bil24.no, (6) one English-language outlet (What Car / Cars.com / Autocar), (7) only then fall back to broader web search, subject to the Sourcing Verification rule above. This checklist is stated once, in `framework/templates/vehicle-onboarding-checklist.md` — not duplicated here — matching ADR-011's own "pointer, not restatement" discipline.

**6. Onboarding Checklist and Dry-Run Tooling.** `framework/templates/vehicle-onboarding-checklist.md` (a new template) and `scripts/onboarding_helpers.py` (a new, read-only helper script — the first code committed to this repository) are introduced. The script is explicitly scoped to sanity-checking `OverallScore`/`CoveragePercent` arithmetic and computing the next available ID for a given worksheet; it must never be written to by, or relied upon by, the workbook itself. The workbook's own formulas remain the sole authoritative calculation, per the existing Calculation Boundaries invariant ("Manual modification of calculated scores is prohibited").

---

# Rationale

None of this is new policy. It is a description of behaviour the last six onboarding sessions already followed consistently — the same framing ADR-010 used for the Framework Versioning Rule. `docs/CONTRIBUTING.md` already states the governing principle directly: "Documentation defines the framework... Data never defines methodology." Confidence defaults, ID conventions, and flagship-Configuration scope have all been methodology in practice for six vehicles running; this ADR simply moves them from data into documentation, where the project's own stated philosophy says they belong.

The sourcing-verification rule is added because the BYD/NAF incident demonstrated a concrete, repeatable failure mode — an AI search summary asserting a specific, plausible-sounding figure that the cited source does not actually contain — that was caught this one time by chance, not by any process guarantee. Given winter-range verification (`M003`) has already been the single most recurring point of friction across all six vehicles onboarded so far, formalizing this check is squarely preventative, not speculative.

---

# Consequences

## Documentation

* `framework/architecture/workbook-schema.md` gains a Confidence Defaults subsection, a stated ID-prefix rule and table (replacing "examples only"), a corrected `ConfigurationID` example, and a new Flagship Configuration Scope section.
* `docs/02_criteria-and-weighting.md`'s Evidence Policy gains a Sourcing Verification subsection.
* No edit to `docs/03_scoring-model.md` or `docs/CONTRIBUTING.md`. Both already carry Confidence/Evidence-Hierarchy content intentionally duplicated at their own altitude; adding the new *operational* defaults there too would compound duplication rather than follow the "pointer, not restatement" discipline ADR-011 established. Each new rule gets exactly one canonical home.

## Workbook

* `11_DecisionLog` gains one new entry (`DEC_ADR_013`).
* No new column, no schema change, no re-scoring triggered.
* `VersionBumpType = NONE`; `WorkbookVersion` and `FrameworkVersion` are unchanged, matching ADR-011's precedent for a pure documentation/process decision.

## Repository

* `framework/templates/vehicle-onboarding-checklist.md` is added.
* `scripts/onboarding_helpers.py` and `scripts/README.md` are added — the first code committed to this repository, which until now was documentation and workbook data only.

---

# Alternatives Considered

## Leave these as informal, undocumented convention

Rejected. Undocumented convention has already cost real onboarding friction this session and cannot be checked by a future contributor, or by any tool, without first reverse-engineering it from six vehicles' worth of live data.

## Fold the Flagship Configuration Scope rule into ADR-009

Rejected. ADR-009 is specifically scoped to `HardRequirementResult` sibling inference. Conflating it with a broader Evidence/Review/Score scoping decision would blur two genuinely distinct rules — the same discipline ADR-011 applied when it declined to fold unrelated Notes cleanups into a single change.

## Build a write-capable script that inserts rows automatically, instead of a read-only dry-run/lookup tool

Rejected for this ADR. The user's own scoping for this change is a read-only sanity-check plus an ID-lookup helper. A write-capable tool is a legitimately larger, separately-scoped follow-on — noted here as deferred, not rejected, mirroring ADR-009's own treatment of the `12_OverallScores.Notes` override-derivation work.

## Require a paired `/audits/ARCH-013` evaluation document

Rejected. This ADR is standalone, not part of the closed P1–P6 structured architecture review — matching the precedent of earlier standalone process ADRs (ADR-003, ADR-004), which were not paired with an audit document.

---

# Migration Strategy

None. This ADR documents already-followed convention; it does not require re-labeling any existing Confidence value, renaming any existing identifier, or retroactively scoring any sibling Configuration. This change is forward-looking only — no existing Vehicle's, Configuration's, or record's data is audited, flagged, or modified for convention conformance as part of this decision.

---

# Impact

Affected documents:

* framework/architecture/workbook-schema.md
* docs/02_criteria-and-weighting.md

Affected workbook:

* 11_DecisionLog (new entry, `DEC_ADR_013`)

New repository artifacts (non-workbook):

* framework/templates/vehicle-onboarding-checklist.md
* scripts/onboarding_helpers.py
* scripts/README.md

No changes are required to:

* docs/CONTRIBUTING.md, docs/01_project-philosophy.md, docs/03_scoring-model.md
* 01_Criteria, 10_Scoring, 12_OverallScores — no criterion weight, type, or calculation-rule change; this ADR triggers no re-scoring

---

# Guiding Principle

> **A convention only the last six sessions remember is not a convention — it is a coincidence waiting to break on the seventh.**
