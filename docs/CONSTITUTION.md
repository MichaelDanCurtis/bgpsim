<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0
- Modified principles:
	- I. Test-First and Contracts (clarified contract-first requirement)
	- V. Observability, Versioning, Simplicity (performance notes referenced)
- Added principles:
	- VI. RFC Compliance & Fidelity
	- VII. Extensible Plugin Architecture
	- VIII. Safety & Isolation by Default
	- IX. Reproducible Scenarios & Seeds
	- X. Performance & Scale Targets
- Added sections: Project Direction & Scope
- Removed sections: None
- Templates requiring updates:
	- ✅ .specify/templates/plan-template.md (previously updated; no new changes)
	- ✅ .specify/templates/spec-template.md (reviewed; aligns)
	- ✅ .specify/templates/tasks-template.md (reviewed; aligns)
- Follow-up TODOs: None
-->

# BGPemu2 Constitution

## Core Principles

### I. Test-First and Contracts (NON-NEGOTIABLE)

TDD is mandatory. Write contract and acceptance tests before implementation; run
Red-Green-Refactor cycles for every change. Public interfaces MUST be specified
via testable contracts (CLI or library) and changes require test updates first.

### II. Library-First Modularity

Each feature starts as a standalone, self-contained library: independently
testable, documented, with a clear purpose. Avoid organization-only libraries;
prefer reuse and small, composable units.

### III. CLI-First, Text I/O

All libraries expose functionality via a CLI. Protocol: stdin/args → stdout;
errors → stderr. Support JSON and human-readable outputs where applicable to
enable automation and easy debugging.

### IV. Integration + Contract Testing

Integration tests are required for: new library contracts, contract changes,
cross-library communication, and shared schemas. Contract tests MUST remain
backward compatible unless a major version is planned with migration notes.

### V. Observability, Versioning, Simplicity

Use structured logging with trace IDs; ensure deterministic, reproducible runs
(seedable randomness, fixed inputs). Follow SemVer; breaking changes require a
migration plan. Start simple (YAGNI) and remove accidental complexity.

### VI. RFC Compliance & Fidelity

Protocol behavior MUST align with relevant RFCs and drafts under test.
Where deliberate deviations are needed (for research), they MUST be explicit,
toggleable, and covered by tests and documentation.

### VII. Extensible Plugin Architecture

Core provides scheduler, topology, eventing, tracing, and I/O. BGP engines
(e.g., FRR, BIRD, GoBGP) and data-plane simulators integrate via stable plugin
interfaces with contract tests per adapter.

### VIII. Safety & Isolation by Default

Emulations MUST run in isolated environments (network namespaces, containers or
VMs). No external advertisements or route leaks. Resource limits must have safe
defaults and be configurable.

### IX. Reproducible Scenarios & Seeds

All scenarios are defined declaratively (DSL/config) and MUST be reproducible
using a single seed and versioned assets. Artifacts include topology, policies,
timing, and randomization controls.

### X. Performance & Scale Targets

Set and track scale goals (e.g., 1k routers, 100k prefixes) and latency goals
for convergence and event propagation. Benchmarks are part of CI gates.

## Additional Constraints

- Determinism: Builds and runs MUST be reproducible across machines.
- Security: No secrets in source; use env or secret stores; enable
  dependency scanning in CI.
- Licensing: All dependencies MUST have compatible licenses and be tracked.
- Formatting/Linting: Enforce consistent formatting and linters in CI.
- Data: Default to immutable inputs/outputs; avoid hidden state.

## Project Direction & Scope

- Upstream Fork: Project originates as a fork of `bgpsim` and will maintain a
	periodic sync with upstream while contributing back improvements when
	applicable and license-compatible.
- Scope Expansion: Beyond control-plane simulation, add adapters for different
	BGP engines and optional data-plane emulation. Provide a scenario DSL, trace
	capture/export (pcap/json), and policy testing harnesses.
- Use-Cases: Education, RFC conformance, regression testing, failure drills,
  performance benchmarking, and research experimentation.

## Development Workflow

- Branches: `[###-feature-name]` convention; small, focused PRs.
- Reviews: Every PR MUST include Constitution Check notes and updated tests.
- CI Gates: Lint, unit, contract, and integration tests MUST pass.
- Releases: Tag with SemVer; changelog includes migration notes for breaking
  changes.
- Documentation: Update specs/plan/tasks alongside code changes.
- Upstream Sync: Track upstream changes; run contract and regression suites
  before merging upstream updates.
- Benchmarks: Maintain performance baselines; regressions block release.

## Governance

- Authority: This constitution governs engineering practices for BGPemu2 and
  supersedes conflicting conventions.
- Amendments: Propose a PR referencing rationale and impact; include version
  bump type and migration considerations; require at least one maintainer
  approval.
- Versioning Policy: Use SemVer for this document. MAJOR for incompatible
  principle changes/removals, MINOR for additions/expansions, PATCH for
  clarifications.
- Compliance: PR reviewers MUST verify compliance; exceptions require written
  justification in the PR and an entry in the plan’s Complexity Tracking.

- Upstream Relationship: Document fork origin, license headers, and sync
	procedures. Prefer upstream-first fixes when feasible. Maintain a CHANGELOG
	section for upstream syncs with notes on divergence.

**Version**: 1.1.0 | **Ratified**: 2025-11-15 | **Last Amended**: 2025-11-15
