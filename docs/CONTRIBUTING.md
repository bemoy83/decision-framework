# Contributing

Thank you for contributing to the **EV Decision Framework**.

This project is built around a simple principle:

> **The framework is more important than any individual evaluation.**

Vehicles, products and scoring models may change over time.

The framework should remain consistent.

---

# Guiding Principle

Contributors should prioritise:

1. Transparency
2. Repeatability
3. Explainability
4. Evidence
5. Long-term maintainability

Every contribution should improve one or more of these principles.

---

# Repository Philosophy

The repository follows a strict separation of responsibilities.

```
Documentation

↓

Framework

↓

Implementation

↓

Data

↓

Reports
```

Documentation defines the framework.

Implementation follows the documentation.

Data never defines methodology.

---

# Before Contributing

Please read the documentation in the following order.

1. `docs/01_project-philosophy.md`
2. `docs/02_criteria-and-weighting.md`
3. `docs/03_scoring-model.md`

These documents define the project architecture.

---

# Evidence First

Whenever possible, contribute evidence rather than conclusions.

Prefer:

> "Motor.no measured 67 dB at 110 km/h."

instead of

> "The vehicle is quiet."

The framework generates conclusions.

Contributors provide evidence.

---

# Sources

Every significant data point should be traceable.

Whenever practical include:

* Source
* Publication date
* Retrieval date
* Confidence

Unsupported claims should never be added.

---

# Unknown Is Acceptable

Do not replace missing information with assumptions.

If reliable information is unavailable:

```
Unknown
```

is the preferred value.

Unknown data is not considered a framework weakness.

Incorrect data is.

---

# Hard Requirements

Do not modify hard requirements without updating the framework version.

Examples include:

* Maximum vehicle length
* Market
* Ownership assumptions

Changing these values creates a new framework version.

---

# Criteria

Criteria should not be added simply because a feature exists.

Every criterion should answer:

> Does this meaningfully influence long-term ownership satisfaction?

If the answer is uncertain, document the discussion before introducing a new criterion.

---

# Weighting

Weights belong to the framework.

Weights do not belong to individual products.

Changing weighting requires:

* discussion
* documentation
* framework version increment

Historical evaluations should remain reproducible.

---

# Evidence Hierarchy

Preferred order:

Tier A

Objective measurements

Tier B

Professional reviews

Tier C

Long-term ownership experience

No single source should determine an evaluation.

---

# Explainability

Every score should be explainable.

If a contributor cannot explain why a score changed, the change should not be merged.

---

# Pull Requests

Pull requests should ideally address one logical change.

Examples:

* Add a new vehicle
* Improve documentation
* Update evidence
* Refactor workbook
* Improve scoring implementation

Avoid mixing unrelated changes.

---

# Architecture Decision Records

Framework decisions should be documented using ADRs.

Examples include:

* Adding a new criterion
* Changing weighting methodology
* Modifying evidence hierarchy
* Introducing new framework concepts

ADRs preserve project history and explain *why* decisions were made.

---

# Coding Principles

Implementation should always remain independent from methodology.

Avoid:

* hardcoded weights
* duplicated data
* hidden calculations
* undocumented assumptions

Prefer:

* configuration
* modularity
* explicit relationships
* reproducible calculations

---

# AI Contributions

AI assistants are welcome contributors.

Recommended responsibilities:

**ChatGPT**

* Framework architecture
* Methodology
* Documentation
* Evidence synthesis
* Decision analysis

**Codex / Cursor**

* Repository maintenance
* Workbook implementation
* Automation
* Refactoring
* Tooling
* Testing

Both should treat the documentation as the source of truth.

---

# Long-Term Vision

This repository is intentionally designed as a general decision framework.

Electric vehicles are the first implementation.

Future domains may include other products requiring structured, evidence-driven decision making.

The framework should evolve without becoming tied to any specific product category.

---

# Final Rule

> **Improve the framework before improving the ranking.**

Better methodology produces better decisions.

Better decisions produce better recommendations.
