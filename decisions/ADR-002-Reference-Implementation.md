# ADR-002 – Reference Implementation

**Status:** Accepted
**Date:** 2026-07-04
**Framework Version:** 1.0

---

# Context

The EV Decision Framework is intended to become a reusable, technology-independent decision framework.

The long-term vision includes support for:

* relational databases;
* REST APIs;
* web applications;
* AI-assisted data collection;
* configurable scoring engines.

However, the project is currently in its first implementation phase.

At this stage, the primary objective is to validate the framework methodology rather than optimise the implementation technology.

---

# Problem

The project requires a concrete implementation that enables:

* framework validation;
* iterative development;
* data collection;
* scoring verification;
* architecture refinement.

Choosing a production-grade technology too early would increase implementation effort before the framework itself has been validated.

---

# Decision

The first implementation of the EV Decision Framework shall be an Excel workbook (`.xlsx`).

The workbook is designated as the **Reference Implementation**.

The workbook exists to implement the framework.

It does not define the framework.

---

# Rationale

The workbook provides several advantages during the initial development phase.

## Accessibility

The workbook can be opened and reviewed without specialised software.

Contributors can inspect data directly.

---

## Transparency

Data remains visible.

Relationships remain understandable.

Calculations can be inspected manually.

This improves framework validation.

---

## Low Implementation Cost

Changes to the framework can be reflected quickly.

Iteration speed is prioritised over implementation sophistication.

---

## Migration Readiness

The workbook schema has been intentionally designed around relational principles.

Migration to:

* SQLite;
* PostgreSQL;
* REST APIs;
* web applications;

should require minimal conceptual changes.

---

# Scope

The Reference Implementation is responsible for:

* storing framework data;
* implementing framework relationships;
* calculating framework scores;
* validating framework rules;
* demonstrating framework behaviour.

It is **not** responsible for:

* defining methodology;
* introducing new framework concepts;
* changing scoring rules.

---

# Implementation Principles

The workbook shall:

* follow the documented workbook schema;
* implement documented entities;
* preserve referential integrity where practical;
* avoid duplicated data;
* remain explainable;
* remain portable.

---

# Limitations

The workbook intentionally accepts several limitations.

Examples include:

* manual data entry;
* limited validation compared to databases;
* limited automation;
* spreadsheet-specific usability constraints.

These limitations are acceptable because they do not affect framework methodology.

---

# Migration Strategy

Future implementations should replace the workbook without requiring framework redesign.

Expected migration path:

```text
Framework Documentation
        │
        ▼
Reference Workbook
        │
        ▼
SQLite / PostgreSQL
        │
        ▼
REST API
        │
        ▼
Web Application
```

Each implementation should preserve:

* framework behaviour;
* scoring methodology;
* traceability;
* explainability.

---

# Responsibilities

The Reference Implementation serves as:

* the validation environment;
* the implementation baseline;
* the migration reference.

It is not intended to become the permanent implementation.

---

# Alternatives Considered

## SQLite

Pros:

* Strong relational integrity.
* Better validation.

Rejected because:

* Increased development overhead.
* Less accessible during early framework development.

---

## PostgreSQL

Pros:

* Production-ready architecture.

Rejected because:

* Premature optimisation.
* Infrastructure requirements outweigh current benefits.

---

## Custom Web Application

Pros:

* Excellent user experience.
* Flexible architecture.

Rejected because:

* The framework itself has not yet stabilised.
* UI development would distract from methodology validation.

---

## Google Sheets

Pros:

* Easy collaboration.

Rejected because:

* Behaviour differs from Excel.
* Less suitable as a stable reference implementation.

May be supported in the future as an additional implementation.

---

# Consequences

Positive:

* Fast iteration.
* Simple collaboration.
* Easy validation.
* Strong alignment with framework documentation.
* Low implementation risk.

Trade-offs:

* Some manual processes remain.
* Spreadsheet limitations must be accepted.
* Advanced validation will be deferred until future implementations.

These trade-offs are considered acceptable during Framework Version 1.x.

---

# Future Direction

The workbook should eventually become one implementation among several.

Future implementations should be evaluated against the workbook during migration to ensure behavioural equivalence.

The framework itself should remain independent of implementation technology.

---

# Related Documents

* README.md
* Implementation Contract
* Workbook Schema
* Entity Model
* Data Flow

---

# Guiding Principle

> **The Reference Implementation validates the framework. It does not define it.**
