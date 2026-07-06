# ADR-001 – Documentation Is the Source of Truth

**Status:** Accepted
**Date:** 2026-07-04
**Framework Version:** 1.0

---

# Context

The EV Decision Framework is intended to evolve over multiple years, potentially across several implementations and technologies.

The first implementation is an Excel workbook, but future implementations may include relational databases, web applications and APIs.

Without a clearly defined source of truth, implementation details risk becoming the de facto specification, causing documentation drift, inconsistent behaviour and implementation-specific assumptions.

The project therefore requires a clear hierarchy of authority.

---

# Decision

The framework documentation is the authoritative source of truth.

Implementations shall conform to the documentation.

Implementations shall never redefine framework behaviour.

If implementation and documentation disagree, the documentation is considered correct until an explicit framework decision updates it.

---

# Documentation Hierarchy

The following order of precedence applies.

## Level 1 – Framework Documentation

Defines methodology and architecture.

Examples:

* Project Philosophy
* Criteria & Weighting
* Scoring Model
* Architecture documents
* Implementation Contract

---

## Level 2 – Architecture Decisions (ADR)

Documents significant architectural decisions that affect framework behaviour.

ADRs explain:

* why a decision was made;
* alternatives considered;
* consequences of the decision.

ADRs complement framework documentation but do not replace it.

---

## Level 3 – Reference Implementation

The Excel workbook is the first reference implementation.

Its purpose is to implement the framework.

It does not define the framework.

---

## Level 4 – Supporting Material

Examples:

* research notes;
* comparison reports;
* temporary analyses;
* implementation discussions.

Supporting material may inform the framework but never override it.

---

# Repository Roles

The project follows a clear separation of responsibilities.

## Product Owner

Responsible for:

* project vision;
* backlog prioritisation;
* repository ownership;
* final acceptance of architectural decisions.

---

## Chief Architect

Responsible for:

* framework methodology;
* architecture;
* documentation;
* design reviews;
* consistency across the framework.

---

## Implementation Engineer

Responsible for:

* implementing the framework;
* refactoring;
* automation;
* tooling;
* validation;
* maintaining compliance with the implementation contract.

Implementation decisions shall remain faithful to the documented framework.

---

# Decision Process

Framework changes follow this sequence:

1. Identify a problem.
2. Discuss alternatives.
3. Update documentation if required.
4. Record significant architectural decisions using an ADR.
5. Update implementations.
6. Verify implementation compliance.

Implementation should never precede architectural agreement.

---

# Consequences

Positive:

* Documentation remains authoritative.
* Implementations remain replaceable.
* Multiple implementations can coexist.
* Framework behaviour stays consistent across technologies.
* Architectural decisions remain traceable.

Trade-offs:

* Documentation requires maintenance.
* Framework changes require discipline.
* Implementers may need to pause work until documentation is updated.

These trade-offs are considered acceptable in exchange for long-term maintainability and reproducibility.

---

# Compliance

An implementation is considered framework compliant only if it follows the documented methodology and architecture.

Passing tests alone is not sufficient if implementation behaviour differs from the documented framework.

---

# Related Documents

* README.md
* CONTRIBUTING.md
* Project Philosophy
* Scoring Model
* Implementation Contract

---

# Guiding Principle

> **The framework defines the implementation. The implementation never defines the framework.**
