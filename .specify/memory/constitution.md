# Samont Constitution

Samont is an industrial ESP32-S3 embedded platform.

## Core Principles

### I. Hardware Interfaces Specified Before Implementation
Hardware interfaces and electrical limits MUST be specified before implementation
begins. Pin assignments, voltage/current limits, signal levels, timing budgets,
and bus configurations MUST be documented in a specification artifact prior to
writing driver or application code against them. Rationale: implementing against
an undocumented or assumed interface risks hardware damage, rework, and silent
incompatibility in an industrial deployment context.

### II. Datasheet Authority
Datasheet requirements take precedence over assumptions. Where a manufacturer
datasheet, errata, or application note specifies a behavior, limit, or timing
requirement, that specification MUST be followed and cited in code comments or
design docs; deviations MUST be explicitly justified and documented. Rationale:
assumptions about component behavior are a leading cause of field failures in
embedded systems.

### III. Explicit Safety and Protection Specification
Safety-critical and protection-related behavior (e.g., overcurrent, overvoltage,
thermal, watchdog, fail-safe states) MUST be explicitly specified, including
trigger thresholds, response actions, and recovery behavior, before
implementation. Undocumented or implicit safety behavior is NOT permitted.
Rationale: protection logic that is not explicitly specified cannot be reviewed,
tested, or certified.

### IV. Measurable Real-Time and Resource Constraints
Real-time behavior and resource constraints (timing deadlines, interrupt
latency, task periods, stack/heap/flash budgets) MUST be stated in measurable,
quantifiable terms and MUST be verifiable through instrumentation, profiling, or
test. Vague qualitative claims (e.g., "fast enough") are NOT acceptable.
Rationale: only measurable constraints can be validated and regressions
detected.

### V. Hardware Abstraction Layer
Hardware abstraction MUST separate application logic from ESP32-specific
drivers. Application code MUST interact with hardware through abstraction
interfaces rather than directly against ESP-IDF or chip-specific APIs.
Rationale: this isolation enables testing without hardware, limits the blast
radius of SoC/driver changes, and supports future hardware migration.

### VI. Protocol Framing, Error Handling, and Versioning
Communication protocols MUST have explicit framing, error handling, and
versioning. Every protocol definition MUST specify message boundaries/framing
rules, error detection and recovery behavior, and a versioning scheme for
backward/forward compatibility. Rationale: unversioned or implicitly framed
protocols are brittle and unsafe to evolve in deployed industrial systems.

### VII. Deterministic and Reproducible Configuration
Configuration MUST be deterministic and reproducible: identical configuration
inputs MUST produce identical build/runtime behavior, and configuration state
MUST be derivable from version-controlled sources rather than ad hoc or
manual steps. Rationale: reproducibility is required for field diagnostics,
audits, and safe rollback.

### VIII. Testability of External Behavior
Every externally visible behavior MUST be testable. Features, protocol
responses, and safety/protection actions MUST be specified such that an
automated or documented manual test can verify them. Behavior that cannot be
observed or verified MUST NOT be considered complete. Rationale: testability
is the mechanism by which specification compliance is confirmed.

### IX. Simplicity and Maintainability
Simple, maintainable designs MUST be preferred over unnecessary abstraction.
Abstraction layers, patterns, and indirection MUST be justified by a concrete
present need, not speculative future flexibility. Rationale: embedded systems
with constrained resources and long service lifetimes are harmed more by
accidental complexity than by occasional duplication.

### X. Specification Traceability
All implementation decisions MUST remain traceable to a specification.
Code, configuration, and protocol behavior MUST be linked back to a written
requirement, datasheet reference, or design document. Implementation without a
traceable origin MUST be flagged and resolved before merge. Rationale:
traceability enables audit, review, and safe long-term maintenance of an
industrial platform.

### XI. Protocol Consistency
Samont shall use the Rembus protocol as the common application-level
communication protocol across all supported transports, including BLE, serial,
Wi-Fi, Ethernet, and other future transports.

CBOR shall be the canonical wire encoding for Rembus messages.

Transport-specific implementations shall transport Rembus messages without
redefining the application-level message semantics.

A new transport shall reuse the existing Rembus protocol rather than
introducing a transport-specific application protocol, unless an explicit
architectural exception is justified and documented.

Rationale: a single application-level protocol with a canonical encoding
prevents semantic drift between transports and keeps transport additions
low-risk.

### XII. Platform and Framework
Samont firmware shall be developed using Espressif's ESP-IDF framework as the
primary firmware development framework.

Samont shall use ESP-IDF APIs and components for hardware, networking, RTOS
integration, security, storage, and other platform functionality where
appropriate.

Platform-specific implementation details shall remain isolated from
application-level logic where practical, allowing the application
architecture to remain independent of individual ESP-IDF components.

Rationale: standardizing on ESP-IDF avoids duplicating vendor-maintained,
certified platform functionality, while isolating platform details keeps the
application layer portable and consistent with the Hardware Abstraction Layer
principle.

## Technology Constraints

- Target platform: ESP32-S3.
- Firmware framework: ESP-IDF.
- Application communication protocol: Rembus.
- Canonical message encoding: CBOR.
- Supported transports include BLE, serial, Wi-Fi and Ethernet.

## Governance

This constitution supersedes all other project practices, coding conventions,
and informal agreements for the Samont platform. Where a conflict exists
between this document and other guidance (READMEs, code comments, prior
practice), this constitution governs.

**Amendment procedure**: Amendments are proposed via a documented change
(e.g., pull request) that updates this file directly, including an updated
Sync Impact Report describing the change. Amendments MUST state the rationale
for the change and identify any principles added, removed, or redefined.

**Versioning policy**: This constitution follows semantic versioning:
- MAJOR: Backward-incompatible governance or principle removals/redefinitions.
- MINOR: New principle or section added, or materially expanded guidance.
- PATCH: Clarifications, wording, or non-semantic refinements.

**Compliance review**: All feature specifications, plans, and implementation
reviews MUST verify compliance with these principles. Any deviation MUST be
explicitly justified in the relevant design artifact and, where it represents
a recurring need, proposed as a constitutional amendment rather than a
one-off exception.

**Version**: 1.0.0 | **Ratified**: 2026-10-07 | **Last Amended**: 2026-10-07
