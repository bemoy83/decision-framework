# ARCH-012 — Source/Evidence Enrichment Evaluation

**Status:** Evaluation Complete — No Implementation Performed
**Date:** 2026-07-07
**Framework Version:** 1.7
**Related:** docs/02_criteria-and-weighting.md Evidence Policy, framework/architecture/enumerations.md, ADR-011

---

# Question Being Answered

Should `09_Sources` gain `PublishedDate`, `RetrievedDate`, and `Language` metadata, ahead of adding more vehicles and possibly automating collection? The user does not propose changing the entity model, only enriching the existing `Source` entity.

---

# Executive Summary

**Two of the three proposed fields already exist — the actual gap is narrower and more specific than it first appears, and `Language` is the one genuinely missing, genuinely useful addition.**

`09_Sources` has carried `PublishedDate` and `RetrievedDate` columns since early in the workbook's history. `RetrievedDate` is fully populated — 87 of 87 Sources. `PublishedDate` is not a schema gap but a **completion** gap: only 14 of 87 rows (16%) have it. The pattern is not random — every `MANUFACTURER`-type Source missing it is a continuously-updated configurator/spec page with no real publish date to record (correctly blank, not a gap). Nearly every `PROFESSIONAL_REVIEW`-type Source missing it is a dated review article that almost certainly *has* a publish date sitting on the page, simply never captured. This is the same category of risk the workbook has already run into once: Tesla's own M003 winter-range investigation had to manually notice that a 2022 test used "a superseded pre-refresh Model 3 generation" — precisely the kind of staleness `PublishedDate` exists to make checkable without re-reading the source.

`Language` does not exist today, and the data shows it would be immediately useful: of 87 Sources, 31 are Norwegian-domain (`.no`) and 56 are not (UK/US review sites, a UK manufacturer spec PDF, one US tech-news source). A framework whose stated Version 2 goal is AI-assisted evidence collection benefits concretely from knowing, per Source, which language extraction/translation risk applies — today that fact is only recoverable by inspecting the URL string, which is exactly the kind of implicit, non-stable-identifier inference the framework's own relationship principles discourage.

A related, older concern from the original P6 flag — that the Evidence Policy's Tier A/B/C source hierarchy is "policy-only, unenforceable" because `09_Sources` has no tier column — turns out to be substantially already answered by the existing `Type` enum (`MANUFACTURER`/`CERTIFICATION` ≈ Tier A, `PROFESSIONAL_REVIEW` ≈ Tier B, `COMMUNITY`/`OWNER_EXPERIENCE` ≈ Tier C), which just needs the mapping written down rather than a new column. Separately, the "direct fetch vs. search-result-synthesis" caveat flagged in the same original note is already a legitimate use of `Source.Notes` under the Notes policy just ratified (ADR-011) — an implementation comment, not something needing its own field.

---

# Current Assessment

**Two Fields Present and Working; One Genuinely Missing.**

| Proposed field | Status | Finding |
| --- | --- | --- |
| `RetrievedDate` | Already exists, fully populated (87/87) | No gap. Already doing exactly the job proposed. |
| `PublishedDate` | Already exists, sparsely populated (14/87, 16%) | Not a schema gap — a completion gap, concentrated specifically in `PROFESSIONAL_REVIEW`-type Sources where the date is usually available and materially useful for judging review recency/staleness. `MANUFACTURER`-type Sources are correctly blank (no real publish date exists for a live configurator page). |
| `Language` | Does not exist | Genuinely missing, genuinely useful: 31/87 Sources are Norwegian-domain, 56/87 are not — a real, substantial split, not a hypothetical one. |

**Carried over from the original P6 flag, resolved without new columns:**

* Tier A/B/C source classification (Evidence Policy) — already substantially expressible via the existing `Type` enum (`MANUFACTURER`, `CERTIFICATION`, `PROFESSIONAL_REVIEW`, `COMMUNITY`, `OWNER_EXPERIENCE`, `GOVERNMENT`, `DATABASE`, `VIDEO`, `OTHER`); needs a documented mapping, not a new field.
* "Direct fetch vs. search-result synthesis" caveat — already a correct use of `Source.Notes` as an implementation comment under the Notes policy ratified in ADR-011; does not need a dedicated column.

---

# Risks

**Risk of adding `Language` now:**

* Minor — one new column, one new small enumeration, no relationship change. Low migration cost: 87 rows to backfill, each answerable directly from the Source's own URL/Publisher (already inspected once to create the row).

**Risk of not adding it:**

* Nothing breaks today, but the value compounds with scale: at 50 vehicles and a proportionally larger Source count, "which sources are in a language other than the target market's" becomes exactly the kind of fact a future automated-collection pass would need to check per-Source rather than re-infer from URLs each time.

**Risk of mandating full `PublishedDate` backfill immediately:**

* Real but bounded effort — roughly 35–40 `PROFESSIONAL_REVIEW` rows would need a publish date looked up. Not urgent enough to block adding a 6th vehicle; better treated as an incremental, ongoing data-quality habit than a one-shot migration, since new Sources should simply capture it going forward.

---

# Recommendation

**Small, targeted enrichment — not a redesign, consistent with the user's own framing.**

1. Add a `Language` column to `09_Sources`, backed by a new small enumeration (e.g. `NB`, `EN`, `OTHER`) reflecting the two languages actually present in the data today plus an escape value. Backfill all 87 existing rows from each Source's own URL/Publisher — a mechanical, low-risk pass.
2. Document the `Type` → Evidence-Policy-Tier mapping in `docs/02_criteria-and-weighting.md`'s Evidence Policy section, closing the original P6 concern without a new column.
3. Do not add a dedicated "fetch method" field — the existing `Source.Notes` already correctly carries this under the Notes policy ratified in ADR-011.
4. Treat `PublishedDate` completion as an ongoing habit (fill it when gathering a new `PROFESSIONAL_REVIEW`-type Source; backfill opportunistically when a Source is next touched for another reason) rather than a forced migration — the column and the need are both already established, only the discipline is missing.

This can proceed as a lightweight follow-up ADR (one column, one enumeration, one documentation mapping), the same pattern used for P1–P5.

---

# Definition of Done

* [x] Verified `09_Sources`'s actual current schema against the three proposed fields (87 rows).
* [x] Quantified `PublishedDate` completion (14/87) and diagnosed the pattern (correctly blank for `MANUFACTURER`, incompletely captured for `PROFESSIONAL_REVIEW`).
* [x] Quantified the real language split (31 `.no` / 56 non-`.no`) supporting the `Language` proposal with data, not assumption.
* [x] Resolved the original P6 Tier A/B/C concern against the live `Type` enum distribution (`MANUFACTURER` 28, `PROFESSIONAL_REVIEW` 52, `OTHER` 4, `COMMUNITY` 2, `CERTIFICATION` 1) rather than proposing a redundant new column.
* [x] Resolved the original P6 fetch-method concern against the Notes policy just ratified in ADR-011.
* [x] Architectural trade-offs documented (Risks section).
* [x] No implementation changes made — `docs/02_criteria-and-weighting.md`, `framework/architecture/enumerations.md`, and `data/EV_Decision_Framework.xlsx` are unmodified.
* [x] A single clear recommendation provided: add `Language` (small, real enrichment); document the `Type`→Tier mapping (no schema change); leave fetch-method to Notes; treat `PublishedDate` completion as an ongoing habit, not a forced migration.
