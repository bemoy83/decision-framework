# EV Decision Framework

## Enumerations

**Version:** 1.0
**Status:** Locked
**Last Updated:** 2026-07-04

---

# Purpose

This document defines all controlled vocabulary used throughout the EV Decision Framework.

Enumerations ensure consistent data entry, improve data quality and simplify future migration to databases and APIs.

Only values defined in this document should be used.

---

# General Principles

Enumerations should be:

* Stable
* Uppercase
* English
* Singular
* Machine-friendly
* Human-readable

Enumeration values should never be translated.

Display names may be localized later.

---

# Availability

Describes equipment availability.

| Value       | Description                        |
| ----------- | ---------------------------------- |
| STANDARD    | Included as standard equipment     |
| OPTIONAL    | Available as optional equipment    |
| PACKAGE     | Included through an option package |
| UNAVAILABLE | Not available                      |
| UNKNOWN     | Information unavailable            |

---

# Confidence

Represents confidence in qualitative information.

| Value   | Description                           |
| ------- | ------------------------------------- |
| HIGH    | Multiple independent reliable sources |
| MEDIUM  | Limited supporting evidence           |
| LOW     | Preliminary information               |
| UNKNOWN | Insufficient information              |

Unknown is preferred over assumptions.

---

# Vehicle Status

Represents the lifecycle status of a vehicle.

| Value        |
| ------------ |
| ANNOUNCED    |
| PREORDER     |
| AVAILABLE    |
| DISCONTINUED |
| CANCELLED    |
| UNKNOWN      |

---

# Configuration Status

Represents the availability of a configuration.

| Value        |
| ------------ |
| AVAILABLE    |
| UPCOMING     |
| DISCONTINUED |
| UNKNOWN      |

---

# Requirement Type

Defines how a criterion is treated.

| Value         |
| ------------- |
| HARD          |
| WEIGHTED      |
| INFORMATIONAL |
| EXCLUDED      |

---

# Source Type

Defines the origin of information.

| Value               |
| ------------------- |
| MANUFACTURER        |
| GOVERNMENT          |
| CERTIFICATION       |
| PROFESSIONAL_REVIEW |
| OWNER_REPORT        |
| COMMUNITY           |
| DATABASE            |
| VIDEO               |
| OTHER               |

---

# Evidence Type

Defines the nature of evidence.

| Value            |
| ---------------- |
| MEASUREMENT      |
| OBSERVATION      |
| SPECIFICATION    |
| REVIEW_SUMMARY   |
| OWNER_EXPERIENCE |
| TEST_RESULT      |

---

# Review Category

Represents qualitative evaluation categories.

| Value              |
| ------------------ |
| DRIVING_EXPERIENCE |
| TECHNOLOGY         |
| SOFTWARE           |
| COMFORT            |
| CABIN_QUALITY      |
| VISIBILITY         |
| EFFICIENCY         |
| PRACTICALITY       |
| VALUE              |
| SAFETY             |

Additional categories may be introduced through framework versioning.

---

# Equipment Category

Represents equipment grouping.

| Value             |
| ----------------- |
| SAFETY            |
| DRIVER_ASSISTANCE |
| INFOTAINMENT      |
| LIGHTING          |
| COMFORT           |
| CONVENIENCE       |
| CHARGING          |
| INTERIOR          |
| EXTERIOR          |

---

# Market

Represents the intended sales market.

| Value  |
| ------ |
| NO     |
| SE     |
| DK     |
| FI     |
| EU     |
| UK     |
| GLOBAL |

---

# Unit

Supported measurement units.

Examples:

| Value     |
| --------- |
| MM        |
| CM        |
| M         |
| KM        |
| KMH       |
| KWH       |
| KWH_100KM |
| KW        |
| NM        |
| KG        |
| L         |
| DB        |

Additional units may be added without changing the framework.

---

# Framework Status

Represents maturity of framework components.

| Value      |
| ---------- |
| DRAFT      |
| REVIEW     |
| LOCKED     |
| DEPRECATED |

---

# Decision Status

Represents implementation progress.

| Value       |
| ----------- |
| PROPOSED    |
| ACCEPTED    |
| IMPLEMENTED |
| SUPERSEDED  |

---

# Naming Convention

Enumeration values are immutable.

Changing an enumeration value is considered a breaking framework change.

If semantics change:

* Introduce a new value.
* Deprecate the old value.
* Preserve historical compatibility.

---

# Guiding Principle

Enumerations define the language of the framework.

Every implementation should use these values exactly as specified.
