# Implementation Plan: Specification-Engineer RFC 9911 (Common YANG Data Types)

## Summary

Transform RFC 9911 ("Common YANG Data Types") into a complete GitHub-tracked Agile backlog of Epics, Features, User Stories, and Use Cases using the Digital Engineering Agentic Pipeline (DEAP) spec-orchestrator skill.

RFC 9911 defines two YANG modules:
- **ietf-yang-types** — Core types: counters, gauges, date/time, identifiers, addresses, strings (~29 typedefs)
- **ietf-inet-types** — IP suite types: IP addresses, domain names, ports, URIs, email (~20 typedefs)

## Pre-requisites

- [ ] Read `.pipeline/constitution.md` (functional layer)
- [ ] Read `skills/spec-orchestrator/SKILL.md` in full
- [ ] Verify `.pipeline/constitution.md`, `skills/`, `rules/`, `scripts/` are tracked by git

## Phase 0: Schema Acquisition & Pre-flight

### 0.1 Download RFC 9911 YANG Schemas

**Files to download:**

1. `ietf-yang-types@2025-12-22.yang` from https://raw.githubusercontent.com/YangModels/yang/main/standard/ietf/RFC/ietf-yang-types%402025-12-22.yang
2. `ietf-inet-types@2025-12-22.yang` from https://raw.githubusercontent.com/YangModels/yang/main/standard/ietf/RFC/ietf-inet-types%402025-12-22.yang

**Target directory:** `schema/`

### 0.2 Run YANG-to-LUI Compiler

```bash
python3 scripts/compile_yang.py --input schema/ietf-yang-types@2025-12-22.yang --output app_flutter/assets/logical-layout.json
```

Note: these are typedef/utility modules (no containers or lists), so the compiler output will be a minimal layout manifest with shared type registry entries.

### 0.3 Bootstrap Tracker Labels

```bash
python3 skills/spec-orchestrator/scripts/bootstrap_tracker_labels.py
```

## Phase 1: Structural Extraction (Worker A — schema-specification-engineering)

**Dispatch:** Fresh context-isolated subagent with `schema-specification-engineering` SKILL.md

**Scope:** Parse both YANG modules:
- `ietf-yang-types` — ~29 typedefs across counter/gauge types, date/time types, identifier types, address types, string types
- `ietf-inet-types` — ~20 typedefs across IP address types, domain/host types, port/protocol types, prefix types, URI/email types

**Expected output:**
- 1-2 Epics (one per module, since each has <=40 leaf nodes and depth 1)
- ~5-8 Features grouped by semantic categories per module
- Each Feature with UML class diagram, acceptance criteria, interface requirements, Logical UI bindings

**Verification gate:** `./skills/spec-orchestrator/scripts/verify_model_coverage.py --spec-only --allow-missing-specs`

## Phase 2: Behavioral Extraction (Worker B — spec-user-story-engineering)

**Dispatch:** Fresh context-isolated subagent with `spec-user-story-engineering` SKILL.md

**Scope:** Derive User Stories from RFC 9911 behavioral text, including:
- Counter wrapping/rollover behavior
- Date/time formatting, canonical forms, timezone handling
- Object identifier validation rules
- IP address scope/zone handling
- Link-local address constraints
- Hostname validation rules (RFC 952, RFC 1123 updates)

**Expected output:** ~8-12 User Stories with BDD scenarios, UML sequence/state diagrams, Required Features Matrix

**Verification gate:** `./skills/spec-orchestrator/scripts/verify_model_coverage.py --spec-only --allow-missing-specs`

## Phase 3: System Interaction Extraction (Worker C — spec-usecase-engineering)

**Dispatch:** Fresh context-isolated subagent with `spec-usecase-engineering` SKILL.md

**Scope:** Extract Use Cases from RFC 9911 operational/deployment context, including:
- Type validation and constraint enforcement scenarios
- Data type selection and derivation scenarios
- Canonical form conversion scenarios

**Expected output:** ~3-5 Use Cases with main success scenarios, alternate flows, state diagrams, Realization Matrices

**Verification gate:** `./skills/spec-orchestrator/scripts/verify_model_coverage.py --spec-only --allow-missing-specs`

## Phase 4: Reconciliation & Verification

```bash
# Model coverage (must show 100%)
python3 skills/spec-orchestrator/scripts/verify_model_coverage.py --spec-only

# Backlog reconciliation
python3 skills/spec-orchestrator/scripts/reconcile_backlog.py

# Lint checks
flutter analyze  # if flutter profile detects sources
```

## Phase 5: Final Reporting

Generate summary with links to all created GitHub issues.

## Governance Constraints (All Phases)

- Strict Role Boundary Lock: spec workers MUST NOT read `.pipeline/profiles/` or codebase files
- Every subagent reads its SKILL.md as step 1
- Max 1 specification item per subagent dispatch
- All Mermaid syntax per `rules/platform-independence.md`
- Platform-independent specs only — no framework references
- `generation_mode: "subagent"` in all YAML frontmatter
- 100% model coverage gate — no schema node left unmapped
