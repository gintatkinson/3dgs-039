# Fix Plan: Replace template placeholders in RFC 9911 Epics

## Scope
2 files: `docs/epics/epic-01-rfc9911-yang-types.md` and `docs/epics/epic-02-rfc9911-inet-types.md`

## Change
Replace every occurrence of the literal text `(semantic linkage justification)` with actual semantic justifications describing why each child item (Use Case, User Story) is linked to the Epic.

## Verification
Run `verify_model_coverage.py --spec-only` — UML compliance gate must pass with zero new violations.
