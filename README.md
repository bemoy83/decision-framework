# EV Decision Framework

A data-driven decision framework for evaluating electric vehicle **configurations** through transparent criteria, evidence-based analysis and long-term ownership priorities.

The framework is designed to answer one question:

> **Which electric vehicle configuration provides the greatest long-term ownership value with the fewest meaningful compromises?**

Unlike traditional vehicle comparisons, this project separates **identity**, **facts**, **evidence**, **evaluation** and **personal priorities** into independent layers, allowing every recommendation to remain transparent, explainable and reproducible.

---

# Project Goals

* Build a repeatable configuration evaluation methodology.
* Minimise long-term ownership regret.
* Keep every recommendation traceable to evidence.
* Separate objective data from subjective preference.
* Produce recommendations that can be explained rather than simply ranked.

The current implementation focuses on compact battery electric vehicles for the Norwegian market, while the framework itself is intentionally designed to be reusable across future decision domains.

---

# Repository Structure

```text
/docs
    01_project-philosophy.md
    02_criteria-and-weighting.md
    03_scoring-model.md

/framework
    /architecture
    /evidence

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

The documentation should be read in the following order.

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

Defines **what** the framework measures.

Topics include:

* Hard requirements
* Weighted criteria
* Informational criteria
* Excluded criteria

---

## 3. Scoring Model

Defines **how** the framework evaluates **Configurations**.

Topics include:

* Evaluation pipeline
* Evidence collection
* Confidence model
* Weighted scoring
* Decision logic

---

# Framework Architecture

The framework separates the shared identity of a vehicle from the purchasable product that is ultimately evaluated.

```text
Vehicle
        │
        ▼
Configuration
        │
        ▼
Evidence
        │
        ▼
Review
        │
        ▼
Scoring
        │
        ▼
Recommendation
```

Vehicle provides shared context.

Configuration is the primary evaluation target.

Every recommendation refers to a Configuration.

---

# Core Principles

The framework is built around the following principles.

* Facts before opinions.
* Long-term ownership over first impressions.
* Unknown is preferable to assumed.
* Every score must be explainable.
* Personal preference should only decide between objectively similar Configurations.

---

# Evidence Policy

The framework relies on multiple independent sources.

Preferred evidence hierarchy:

1. Objective measurements and official specifications.
2. Professional automotive reviews.
3. Long-term ownership experience.

No recommendation should rely on a single source.

Every Review shall be supported by documented Evidence.

Every Evidence record shall reference one or more Sources.

---

# Project Workflow

The evaluation workflow is identical for every Configuration.

```text
Vehicle
        │
        ▼
Configuration
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
Review
        │
        ▼
Criterion Scoring
        │
        ▼
Overall Score
        │
        ▼
Recommendation
```

Vehicle supplies shared identity and shared information.

Configuration is the object that passes through the evaluation pipeline.

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

Changes to methodology, criteria or weighting create a new Framework Version.

Configuration evaluations remain bound to the Framework Version under which they were performed.

Historical evaluations remain reproducible.

---

# Long-Term Vision

The project is intentionally developed in stages.

### Version 1

Reference Workbook implementing the framework.

### Version 2

AI-assisted evidence collection, validation and maintenance.

### Version 3 (Optional)

Database-backed decision engine with configurable weighting, advanced filtering and interactive analysis.

---

# Status

**Framework Version:** 1.1 (Architecture Synchronization)

The documentation defines the framework.

The Reference Workbook implements the framework.

Future implementations should preserve the documented methodology, ownership model and traceability while remaining free to choose different technologies.
