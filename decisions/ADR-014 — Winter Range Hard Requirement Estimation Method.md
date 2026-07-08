# ADR-014 — Winter Range Hard Requirement Estimation Method

**Status:** Accepted

**Date:** 2026-07-08

**Framework Version:** 1.9 (Next)

**Supersedes:** None

**Related:**

* ADR-009 (Hard Requirement Result Model)
* ADR-010 (Framework Versioning Rule)
* DEC_M003_VERIFY, DEC_M002_VIOLATION
* docs/02_criteria-and-weighting.md
* 14_HardRequirementResults, 04_Technical, 03_Configurations

---

# Context

M003 (winter motorway range, ~300 km) has been `UNKNOWN` for every one of the six flagship Configurations since it was first checked (`DEC_M003_VERIFY`). Only two real, independently-measured winter highway range data points exist in the entire workbook: Volvo EX30 P8 AWD (338 km, elbil.no) and a Renault 4 test that used the wrong trim (287 km, Iconic tested rather than Techno). Every other Configuration has no independent winter test at all — most of these are 2026 model-year cars too new to have gone through a Norwegian winter test cycle yet.

The user's actual requirement behind M003 is narrower than "300 km with zero charging stops": it is "reach my father's home (the Oslo–Grimstad/E18 route the criterion was originally scoped around) in winter without the trip becoming impractical" — and the user has separately confirmed that needing exactly one charging stop is acceptable, not an automatic disqualifier. Leaving M003 `UNKNOWN` indefinitely — waiting for real winter tests that may not exist before a purchase decision has to be made — was judged worse than a documented, conservative estimate. Whether M003's own threshold or its binary PASS/FAIL shape should change further (to reflect "≤1 stop" rather than "no charging concern") is a separate, larger redesign question, deliberately out of scope for this ADR — see Alternatives Considered.

The two real data points give an empirical anchor for how much of WLTP range survives Norwegian winter highway driving: 338/450 = 75% (Volvo) and 287/398 = 72% (Renault, different trim). The user chose **70% of WLTP range** as the estimation factor — slightly more conservative than either measured ratio, appropriate given M003 gates eligibility rather than just informing a score.

---

# Decision

**Where no independent winter highway range test exists (or existing tests were already rejected as unreliable/mismatched), M003 is now evaluated against an estimate: `WLTP_RANGE × 0.70`, compared to the ~300 km requirement.**

Results recorded in `14_HardRequirementResults` at `Confidence = LOW` (an estimate, not a measurement) with a `Reason` citing the calculation and its source WLTP figure. This replaces the prior `UNKNOWN` determination for all six flagship Configurations:

| Configuration | WLTP | × 0.70 | Result | Margin |
|---|---|---|---|---|
| Tesla Model 3 Long Range RWD | 691 km (19", conservative of two recorded figures) | 483.7 km | PASS | +183.7 km |
| Renault 4 Techno | 398 km | 278.6 km | **FAIL** | −21.4 km |
| Škoda Elroq Selection 60 | 448 km | 313.6 km | PASS | +13.6 km |
| Volvo EX30 P5 Long Range | 475 km | 332.5 km | PASS | +32.5 km |
| Kia EV2 FWD Long Range GT-Line | 453 km | 317.1 km | PASS | +17.1 km |
| BYD Atto 3 Design | 420 km | 294.0 km | **FAIL** | −6.0 km |

Renault's estimate is corroborated by its existing real (if trim-mismatched) test point — 287 km measured vs. 278.6 km estimated, both below 300 km, reinforcing rather than contradicting the FAIL. Volvo's P5 Long Range estimate (332.5 km) is corroborated similarly by the real P8 AWD test (338 km) on the same platform. Tesla's M003 result is estimated the same way for consistency even though its real (rejected-as-unreliable) tests remain on record — Tesla's overall eligibility is unaffected either way since it is already excluded/overridden on M002.

**Per the user's explicit direction, both new FAILs are handled exactly like Tesla's existing M002 FAIL: kept in the comparison, scored, and flagged rather than excluded.** `03_Configurations.HardRequirementOverride` is set `TRUE` for `RENAULT_4_TECHNO` and `BYD_ATTO3_DESIGN` (previously `FALSE`).

A new `04_Technical` row (`TF_WINTER_HIGHWAY_RANGE_ESTIMATE`, `Qualifier = "Estimated (70% of WLTP, ADR-014)"`, `Confidence = LOW`) is added for all six flagships, citing the same Source already used for that Configuration's `TF_WLTP_RANGE` fact. Existing real-test-based `TF_WINTER_HIGHWAY_RANGE_ESTIMATE` rows (Volvo P8 AWD, Renault Techno) are left untouched — the calculated estimate is additive, disambiguated by `Qualifier`, not a replacement of real measurements.

This is a **Methodology**-type Framework Version bump per ADR-010: it changes how a Hard Requirement's PASS/FAIL is *determined* (introducing an estimation rule where previously only direct measurement was accepted), even though the 300 km threshold itself is unchanged. `FrameworkVersion` 1.8 → 1.9. `WorkbookVersion` is unaffected (no new worksheet, column, or enumeration).

---

# Rationale

An estimate anchored to two real Norwegian winter-test ratios (72–75% of WLTP) and rounded down to a more conservative 70% is a defensible middle ground between "wait indefinitely for data that may never arrive before a purchase decision" and "assume the best case." Recording it at `LOW` confidence, with the calculation shown in `Reason`, keeps the framework's core discipline intact — the estimate is visibly an estimate, not laundered into a false-confidence PASS/FAIL, and remains fully traceable to the WLTP source it was derived from.

Treating the two new FAILs identically to Tesla's existing M002 override — user decides per-car whether to keep-and-flag or exclude, rather than the framework silently deciding — is the same precedent already established in `DEC_M002_VIOLATION` and is now applied consistently rather than reinvented per case.

---

# Consequences

## Documentation

* `docs/02_criteria-and-weighting.md`'s Hard Requirements table gains a note that Winter Range may be evaluated via a documented WLTP-based estimate when no independent test exists, citing this ADR.

## Reference Workbook

* `14_HardRequirementResults`: 6 existing M003 rows updated in place (`Result`, `Confidence` where changed, `Reason`, `FrameworkVersion`) — `HardRequirementResultID`s unchanged.
* `03_Configurations.HardRequirementOverride`: `RENAULT_4_TECHNO` and `BYD_ATTO3_DESIGN` set `TRUE`; matching `Notes` pointer added for both, mirroring the existing Tesla wording.
* `04_Technical`: 6 new `TF_WINTER_HIGHWAY_RANGE_ESTIMATE` rows added (`TECH_000160`–`TECH_000165`), one per flagship Configuration.
* `15_Dashboard`: Decision Summary gains a `Hard Requirement Reason` column so a FAIL's actual justification is visible without opening the Configuration Comparator.
* README: `FrameworkVersion` 1.8 → 1.9.

## No changes to

* `01_Criteria` — M003's own Weight (0), Type (HARD), or the ~300 km threshold are unchanged; only how PASS/FAIL is *derived* when no real test exists has changed.
* Any Review, Score, or OverallScore — this ADR touches Hard Requirement eligibility data only, not weighted scoring.

---

# Alternatives Considered

## Redefine M003 itself — from a binary "no charging stop" gate to something reflecting "at most one stop is acceptable"

This is very likely the *more correct* long-term fix — the user explicitly said the underlying need tolerates one stop, which a flat 300 km/no-stop threshold does not represent. Deliberately not done here: it requires knowing the real one-way trip distance (the M003 criterion text already names Oslo–Grimstad/E18, but the exact km figure and a real charging-stop-duration tolerance haven't been established), and changing a Hard Requirement's actual pass/fail *shape* (not just its estimation method when data is missing) is a bigger decision than today's data gap fix. Flagged as a follow-up, not resolved by this ADR.

## Use the higher end of the two real ratios (75%, from Volvo) rather than 70%

Rejected per the user's explicit choice — M003 gates eligibility, not just a score, so erring conservative (70%, below both real observed ratios of 72–75%) was preferred over optimistic rounding.

## Leave M003 `UNKNOWN` indefinitely until real winter tests exist for each car

Rejected — the user needs to make a purchase decision on a timeline that will not wait for a full Norwegian winter test cycle on 2026 model-year cars. An estimate at `LOW` confidence, clearly labeled as such, was judged more useful than a permanent blank.

---

# Guiding Principle

> **An estimate that shows its work and admits its confidence level is more honest than a blank cell waiting for data that may never come — as long as it never gets mistaken for a measurement.**
