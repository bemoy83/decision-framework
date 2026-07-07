# scripts/

Read-only helper tools for onboarding a new Vehicle into the Reference Workbook (ADR-013). Requires `openpyxl` (`pip install openpyxl`). Never writes to `data/EV_Decision_Framework.xlsx`.

## `onboarding_helpers.py`

```bash
# Next available ID for a numeric-sequence worksheet
python3 scripts/onboarding_helpers.py next-id 04_Technical
# -> TECH_000160

# 11_DecisionLog uses a category-tagged ID, so pass --category
python3 scripts/onboarding_helpers.py next-id 11_DecisionLog --category DATA
# -> DEC_DATA_011

# Recompute OverallScore/CoveragePercent for a Configuration, without needing
# Excel/LibreOffice to recalculate the workbook's live formulas
python3 scripts/onboarding_helpers.py score BYD_ATTO3_DESIGN
# -> per-criterion breakdown, then OverallScore/CoveragePercent totals

# Compare that recomputation against the live 12_OverallScores row, if it has
# already been recalculated at least once in Excel/LibreOffice
python3 scripts/onboarding_helpers.py validate BYD_ATTO3_DESIGN
```

`09_Sources`, `02_Vehicles`, and `03_Configurations` use free-text identifiers and are out of scope for `next-id` — pick a name manually, following `framework/architecture/workbook-schema.md`'s Naming Conventions table.

`score` does not replicate `12_OverallScores.Notes`' hardcoded eligibility-override text (e.g. a Tesla-style "HARD REQUIREMENT FAILED" string) — check that column manually.
