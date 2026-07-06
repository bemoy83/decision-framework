# Codex Issue Template

This template defines how implementation tasks are assigned to the Implementation Engineer (Codex).

The EV Decision Framework follows a documentation-first architecture.

Documentation is the source of truth.

Implementations must conform to the framework documentation.

---

# Role

You are acting as the **Implementation Engineer** for the EV Decision Framework.

Your responsibility is to implement documented framework behaviour.

You are **not** responsible for defining methodology or modifying framework architecture.

---

# Definition of Done

This issue is considered complete only if:

- All acceptance criteria have been satisfied.
- The implementation complies with the documented framework.
- Validation has been completed.
- No undocumented assumptions remain.
- A Pull Request has been opened.
- Architecture review has been requested.
- The Pull Request has **not** been merged.

---

# Task

**Issue Number**

<issue-number>

**Title**

<issue-title>

**Objective**

Describe the implementation goal.

Focus on implementation only.

---

# Documentation

Before making changes, read the following documents.

## Repository

* README.md
* CONTRIBUTING.md

## Framework Documentation

* docs/01_project-philosophy.md
* docs/02_criteria-and-weighting.md
* docs/03_scoring-model.md

## Architecture

* framework/architecture/workbook-schema.md
* framework/architecture/entity-model.md
* framework/architecture/relationships.md
* framework/architecture/enumerations.md
* framework/architecture/data-flow.md
* framework/architecture/implementation-contract.md

## Architecture Decisions

Read every accepted ADR before implementation.

---

# Scope

Implement only the requested issue.

Avoid unrelated improvements.

Avoid refactoring outside the issue scope.

---

# Rules

You may:

* modify implementation files;
* improve implementation quality;
* add validation where appropriate.

You may not:

* modify framework methodology;
* modify accepted ADRs;
* invent new entities;
* invent new relationships;
* invent new terminology;
* redesign the framework.

If the documentation appears inconsistent, stop and report the issue.

Do not guess.

---

# Git Workflow

Create a dedicated feature branch.

Recommended naming:

```text
feature/<short-description>
```

Implement the requested changes.

Create logical commits with descriptive commit messages.

Push the branch.

Open a Pull Request against `main`.

Do not merge the Pull Request.

Wait for architecture review.

---

# Validation

Before opening the Pull Request, verify:

* Workbook complies with Workbook Schema.
* Entity names match Entity Model.
* Relationships follow Relationships Model.
* Enumerations follow Enumerations.
* Data Flow has not been violated.
* Implementation Contract has been satisfied.
* No accepted ADR has been violated.

---

# Pull Request

Use the following structure.

## Summary

Brief description of the implementation.

---

## Files Changed

List modified files.

---

## Framework Compliance

Confirm compliance with:

* Workbook Schema
* Entity Model
* Relationships
* Enumerations
* Data Flow
* Implementation Contract
* Accepted ADRs

---

## Validation Performed

Describe how the implementation was verified.

---

## Assumptions

List every assumption made.

If none:

> None.

---

## Documentation Ambiguities

Describe any inconsistencies discovered.

If none:

> None.

---

## Recommended Follow-up Issues

List implementation improvements that are outside the scope of the current issue.

Do not implement them.

---

# Completion Criteria

The task is complete when:

* implementation is finished;
* validation passes;
* the branch has been pushed;
* a Pull Request has been opened;
* architecture review is requested.

The task is **not** complete when implementation has merely been committed locally.

---

# Guiding Principle

> **Implement the documented framework faithfully. Improve implementation quality without changing framework behaviour.**
