# EV Decision Framework

A data-driven decision framework for evaluating electric vehicles through transparent criteria, evidence-based analysis and long-term ownership priorities.

The framework is designed to answer one question:

> **Which electric vehicle provides the greatest long-term ownership value with the fewest meaningful compromises?**

Unlike traditional vehicle comparisons, this project separates **facts**, **evidence**, **evaluation** and **personal priorities** into independent layers, allowing recommendations to remain transparent and explainable.

---

# Project Goals

* Build a repeatable vehicle evaluation methodology.
* Minimise long-term ownership regret.
* Keep every recommendation traceable to evidence.
* Separate objective data from subjective preference.
* Produce recommendations that can be explained rather than simply ranked.

The current implementation is focused on evaluating compact electric vehicles for the Norwegian market, but the framework itself is intentionally designed to be reusable.

---

# Repository Structure

```text
/docs
    01_project-philosophy.md
    02_criteria-and-weighting.md
    03_scoring-model.md

/data
    EV_Decision_Framework.xlsx

/research
    <manufacturer and model research>

/reports
    <generated comparison reports>

/decisions
    <Architecture Decision Records (ADR)>
```

---

# Documentation

The project documentation should be read in the following order.

## 1. Project Philosophy

Defines the principles behind the framework.

Topics include:

* Design philosophy
* Long-term ownership
* Evidence-first approach
* Explainability
* Decision principles

---

## 2. Criteria & Weighting

Defines what the framework measures.

Topics include:

* Hard requirements
* Weighted criteria
* Informational criteria
* Excluded criteria

---

## 3. Scoring Model

Defines how the framework evaluates vehicles.

Topics include:

* Evaluation pipeline
* Evidence collection
* Confidence model
* Weighted scoring
* Decision logic

---

# Framework Architecture

The framework consists of four logical layers.

```text
Vehicle Data
        │
        ▼
Evidence
        │
        ▼
Evaluation
        │
        ▼
Recommendation
```

Each layer is independent from the others.

This separation ensures that:

* new evidence improves evaluations without rewriting historical data;
* weighting changes do not modify technical data;
* recommendations remain reproducible.

---

# Core Principles

The framework is built around the following principles:

* Facts before opinions.
* Long-term ownership over first impressions.
* Unknown is preferable to assumed.
* Every score must be explainable.
* Personal preference should only decide between objectively similar candidates.

---

# Evidence Policy

The framework relies on multiple independent sources.

Preferred evidence hierarchy:

1. Objective measurements and official specifications.
2. Professional automotive reviews.
3. Long-term ownership experience.

No recommendation should rely on a single source.

---

# Project Workflow

The evaluation process follows the same workflow for every vehicle.

```text
Candidate Vehicle
        │
        ▼
Hard Requirements
        │
        ▼
Technical Data
        │
        ▼
Evidence Collection
        │
        ▼
Quality Assessment
        │
        ▼
Weighted Scoring
        │
        ▼
Final Recommendation
```

---

# Current Scope

Current framework assumptions include:

* Battery electric vehicles (BEV)
* Norwegian market
* Maximum vehicle length of 4500 mm
* Long-term ownership (approximately 8–10 years)
* Evidence-based decision making
* Explainable recommendations

---

# Versioning

The framework is version controlled.

Changes to methodology, criteria or weighting create a new framework version.

Vehicle evaluations remain tied to the framework version under which they were performed.

---

# Long-Term Vision

The project is intentionally developed in stages.

### Version 1

Structured Excel workbook implementing the framework.

### Version 2

AI-assisted data collection, validation and maintenance.

### Version 3 (Optional)

Database-backed decision engine with advanced filtering, configurable weighting and interactive analysis.

---

# Status

**Framework Version:** 1.0 (In Development)

The documentation defines the architecture.

The Excel workbook is the reference implementation.

Future automation should build upon these foundations rather than replace them.
