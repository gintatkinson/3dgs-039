# Implementation Plan: Bulk Alternate Flows for RFC 9179 Use Cases

## Summary
Add alternate/exception flows to two use case files to satisfy constraint-to-flow parity rules.

## Work Packages

### WP-1: UC-07 — Add 49 Alternate Flows (5i through 5bf)
**File:** `docs/use-cases/uc-07-rfc9179-geo-register-geo-location-entity.md`
**Current state:** 8 flows (5a–5h), before `## 6. Postconditions` at line 73
**Target:** 57 total flows
**Action:** Append 49 new flows between existing flow 5h (line 72) and the `## 6. Postconditions` section header (line 74). Each flow follows the established format with heading, branch point, 2 numbered steps, and a guarantee statement. Flows will cover:
- Decimal precision edge cases (latitude at boundary, exactly 16 fraction-digits, exactly 6 for height)
- Coordinate range boundaries (-90.0, +90.0, -180.0, +180.0 exactly)
- Choice exclusivity edge cases (empty both cases, switching cases mid-entry)
- Geodetic-datum registry lookup failures (timeout, empty, stale cache)
- Astronomical-body naming edge cases (Mars, Moon, Ceres, mixed-case, trailing spaces, Unicode)
- Invalid coordinate system — missing required leaves within a case
- Height-only without lat/long (missing mandatory ellipsoid leaves)
- Coord-accuracy and height-accuracy override validation
- Velocity validation edge cases (zero vector, negative speeds, single component populated, precision overflow)
- Timestamp format deviations (no timezone, milliseconds precision mismatch)
- Valid-until equality edge case (== timestamp)
- Alternate-system interactions (feature off but leaf empty, feature on but leaf missing, feature on with value)
- Reference-frame inheritance from parent entity
- IANA registry returns deprecated datum
- Simultaneous multiple validation errors
- Empty geo-location container submitted

### WP-2: UC-09 — Add 30 Alternate Flows (5g through 5bk)
**File:** `docs/use-cases/uc-09-rfc9179-geo-validate-geo-location-instance.md`
**Current state:** 6 flows (5a–5f), before `## 6. Postconditions` at line 62
**Target:** 36 total flows
**Action:** Append 30 new flows between existing flow 5f (line 61) and the `## 6. Postconditions` section header (line 63). Same format. Flows will cover:
- Reference-frame resolution edge cases (geodetic-datum missing on non-earth body, datum registry timeout, empty astronomical-body)
- Decimal precision: exactly-at-ceiling, velocity fr12 overflow, height fr6 overflow
- Choice validation: zero-populated cases, all leaves empty in active case, mandatory leaf missing
- Timestamp/valid-until: missing timezone, valid-until == timestamp, future timestamp too far ahead
- Feature-guard: `elif-feature` guard resolution, feature-gated container present without feature
- Coord-accuracy override: accuracy leaf present without datum lookup
- Height-accuracy override: accuracy out of datum capability bounds
- Velocity: single-component present, all-zero vector, negative v-up
- Partial payload: only reference-frame populated, only location populated
- Multiple simultaneous constraint violations
- Empty geo-location container
- ISO 6709 conformance checks for coordinate representation

### WP-3: Commit and Push
```bash
git add docs/use-cases/uc-07-rfc9179-geo-register-geo-location-entity.md docs/use-cases/uc-09-rfc9179-geo-validate-geo-location-instance.md
git commit -m "fix: add bulk alternate flows to meet constraint-to-flow parity (RFC 9179)"
git push origin main
```

### WP-4: Verification
Run `python3 skills/spec-orchestrator/scripts/verify_model_coverage.py --spec-only` and report output.

## Writing Conventions
All flows follow the existing format:
```
- **5x. [Brief condition description] (Branches from Basic Flow step [N]):**
  1. [Actor/System] does [Action].
  2. [Actor/System] does [Action] and returns to step [Y] of the Main Success Scenario / notifies [Actor] of [Result].
```
