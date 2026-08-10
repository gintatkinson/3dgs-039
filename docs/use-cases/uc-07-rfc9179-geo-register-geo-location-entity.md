---
title: "[RFC9179-GEO] Register Geo-Location Entity With Coordinate System"
type: "use-case"
generation_mode: "subagent"
spec_source: "RFC 9179"
schema_containers:
  - path: "geo:geo-location"
    node_type: container
---

# Use Case: [RFC9179-GEO] Register Geo-Location Entity With Coordinate System

## Parent Epic
- [ ] #42 - [RFC9179-GEO A YANG Grouping for Geographic Locations](https://github.com/gintatkinson/3dgs-039/blob/main/docs/epics/epic-04-rfc9179-geo-location.md) (semantic linkage: this use case exercises the full geo-location grouping container — reference-frame resolution, coordinate system choice selection, coordinate value entry, velocity vector recording, and temporal attribute assignment — as defined by the ietf-geo-location YANG module in RFC 9179)

## 1. Actors
- **Primary Actor:** Network Operator
- **Secondary Actors:** IANA Geodetic Registry (provides valid geodetic-datum values for astronomical-body lookup and datum capability metadata)

## 2. Preconditions
- The ietf-geo-location YANG module is loaded and its grouping definitions are available for data instance validation.
- A target managed entity (router, data center, fiber endpoint) exists in the inventory to which a geographic location shall be assigned.
- The IANA Geodetic System Values Registry is reachable to provide valid geodetic-datum-to-astronomical-body mappings and accuracy capability data.
- The alternate-systems feature flag state is known (enabled or disabled) for the application instance.

## 3. Trigger
Network Operator needs to assign a geographic location to a managed entity.

## 4. Main Success Scenario (Basic Flow)
1. Network Operator initiates geo-location registration for the target managed entity.
2. System presents the Reference Frame section with astronomical-body defaulting to "earth" and geodetic-datum defaulting to "wgs-84".
3. Operator inspects the default reference frame values; System resolves the geodetic-datum against the IANA Geodetic Registry to obtain datum capability metadata including default coordinate accuracy and height accuracy bounds.
4. System presents the coordinate system choice: Ellipsoidal (latitude/longitude/height) or Cartesian (x/y/z meters).
5. Operator selects the desired coordinate system and enters coordinate values (latitude and longitude in decimal degrees with up to 16 fraction-digits, and height in meters with up to 6 fraction-digits; or x, y, z in meters with up to 6 fraction-digits).
6. System validates each coordinate value against the reference-frame context: latitude against -90.0 to 90.0 decimal degree standard range, longitude against -180.0 to 180.0 decimal degree standard range, all values against their decimal64 fraction-digit precision limits, and coordinate system membership against the active choice case.
7. Operator optionally enters a velocity vector consisting of v-north, v-east, and v-up components in meters per second with up to 12 fraction-digit precision.
8. Operator sets the timestamp to the current date-and-time capturing when the location was recorded.
9. Operator optionally sets a validity window by specifying a valid-until date-and-time value that is chronologically after the timestamp.
10. System persists the complete geo-location entity — reference frame, coordinate values, velocity vector, timestamp, and validity window — and associates it with the target managed entity, then returns a confirmation with the registered location identifier to the Operator.

## 5. Alternate and Exception Flows
- **5a. Invalid Latitude Range (Branches from Basic Flow step 6):**
  1. System detects the entered latitude value is outside the standard -90.0 to 90.0 decimal degree range applicable to Earth-based and most IAU astronomical body geodetic datums.
  2. System highlights the latitude field with an error indicator, displays a warning that the value exceeds the expected range for the active geodetic-datum, and returns the Operator to step 5 of the Main Success Scenario without preventing re-entry of the same value (the constraint is informational, not a hard schema rejection — the geodetic-datum defines ultimate validity).

- **5b. Invalid Longitude Range (Branches from Basic Flow step 6):**
  1. System detects the entered longitude value is outside the standard -180.0 to 180.0 decimal degree range applicable to Earth-based and most IAU astronomical body geodetic datums.
  2. System highlights the longitude field with an error indicator, displays a warning that the value exceeds the expected range for the active geodetic-datum, and returns the Operator to step 5 of the Main Success Scenario.

- **5c. Coordinate System Mismatch — Ellipsoid Values Submitted with Cartesian Choice Active (Branches from Basic Flow step 5):**
  1. System detects that the Operator has selected the Cartesian coordinate case but is submitting ellipsoid coordinate values (latitude, longitude, height) — or selected the Ellipsoidal case but is submitting Cartesian values (x, y, z).
  2. System rejects the submission, highlights the mismatched coordinate fields, displays a validation error stating that the coordinate values do not match the active coordinate system choice, and returns the Operator to step 4 to re-select the correct coordinate system.

- **5d. Missing Reference Frame with Alternate System (Branches from Basic Flow step 2):**
  1. System detects that the alternate-system leaf is populated (e.g., "simulation-vr-1") but the astronomical-body leaf is unspecified, resulting in an alternate coordinate system definition with no base astronomical body to which it applies.
  2. System rejects the reference-frame configuration, displays a validation error indicating that an alternate-system requires an explicit astronomical-body to define the base frame of reference, and returns the Operator to step 2 to complete the reference-frame fields before proceeding.

- **5e. Accuracy Exceeds Datum Capability (Branches from Basic Flow step 3):**
  1. System detects that the Operator-specified coord-accuracy or height-accuracy value claims higher precision (smaller numeric value) than the geodetic-datum is capable of providing, as indicated by the IANA Geodetic Registry datum metadata.
  2. System displays a non-blocking warning alongside the accuracy field indicating that the specified accuracy exceeds the datum's inherent precision capability, preserves the Operator-entered value, and allows the Operator to continue with step 4 or return to step 3 to adjust.

- **5f. Expired Valid-until Before Timestamp (Branches from Basic Flow step 9):**
  1. System detects that the valid-until date-and-time value is chronologically earlier than the timestamp value, creating a logically impossible validity window where the location expires before it was recorded.
  2. System rejects the valid-until value, highlights the valid-until field with an error indicator, displays a validation error that the validity window end must be after the timestamp, and returns the Operator to step 9 to enter a corrected valid-until value.

- **5g. Invalid Astronomical-Body or Geodetic-Datum Character Format (Branches from Basic Flow step 2 or 3):**
  1. System detects that the astronomical-body or geodetic-datum string value contains control characters or characters outside the permitted ASCII printable ranges (decimal 32-64 and 91-126), violating the schema regex pattern `[ -@\[-\^_-~]*`.
  2. System rejects the value, highlights the offending field with an error indicator, displays a validation error specifying the invalid character range, and returns the Operator to the field for correction.

- **5h. Coordinate Decimal Precision Exceeded (Branches from Basic Flow step 6):**
  1. System detects that one or more coordinate values exceed their defined decimal64 fraction-digit precision limits: latitude or longitude exceeds 16 fraction-digits, height exceeds 6 fraction-digits, or x/y/z exceeds 6 fraction-digits.
  2. System rejects the submission, highlights each field exceeding its precision bound, displays the applicable fraction-digit limit (16 for latitude/longitude, 6 for height and Cartesian coordinates), and returns the Operator to step 5 to enter values with the correct precision.

## 6. Postconditions (Guarantees)
- **Success Guarantee:** A fully validated geo-location entity is persisted in the system, associated with the target managed entity. The entity carries a resolved reference frame (with astronomical-body, geodetic-datum, and accuracy overrides), exactly one active coordinate case (ellipsoid or Cartesian) with validated coordinate values at the correct decimal precision, an optional velocity vector, a timestamp recording when the location was measured, and an optional validity window. The IANA Geodetic Registry lookup has been cached for the active geodetic-datum.
- **Failure Guarantee:** No partial geo-location entity is persisted. If any validation step fails, all entered data is discarded or rolled back to the state prior to the registration attempt. The target managed entity remains without an associated geographic location. The Operator receives a specific validation error message identifying the failing field and constraint.

## UML Diagrams
### Use Case Diagram
```mermaid
graph TD
    subgraph "System Boundary"
        UC_Register(["Register Geo-Location Entity"])
        UC_ResolveDatum(["Resolve Geodetic Datum"])
        UC_ValidateCoords(["Validate Coordinate Values"])
        UC_SetVelocity(["Set Velocity Vector"])
        UC_SetTemporal(["Set Temporal Attributes"])
    end
    NetworkOperator((Network Operator)) --- UC_Register
    UC_Register -. "&lt;&lt;include&gt;&gt;" .-> UC_ResolveDatum
    UC_Register -. "&lt;&lt;include&gt;&gt;" .-> UC_ValidateCoords
    UC_Register -. "&lt;&lt;extend&gt;&gt;" .-> UC_SetVelocity
    UC_Register -. "&lt;&lt;extend&gt;&gt;" .-> UC_SetTemporal
    UC_Register --- IANARegistry((IANA Geodetic Registry))
    UC_ResolveDatum --- IANARegistry
```

### State Machine Diagram
```mermaid
stateDiagram-v2
    [*] --> ReferenceFrameEntry : "initiateGeoRegistration [target entity exists] / presentDefaults"
    ReferenceFrameEntry --> ReferenceFrameResolved : "resolveDatum [astronomicalBody default earth, geodeticDatum default wgs-84] / queryIanaRegistry"
    ReferenceFrameResolved --> CoordinateSystemChoice : "acceptFrame [datum resolved] / presentCoordinateChoice"
    CoordinateSystemChoice --> EllipsoidEntry : "select [ellipsoid] / displayLatLonHeight"
    CoordinateSystemChoice --> CartesianEntry : "select [cartesian] / displayXYZ"
    EllipsoidEntry --> CoordinatesValidated : "enterValues [latitude longitude height] / validateRangeAndPrecision"
    CartesianEntry --> CoordinatesValidated : "enterValues [x y z] / validateRangeAndPrecision"
    CoordinatesValidated --> VelocityEntry : "acceptCoordinates [all valid] / presentVelocitySection"
    CoordinatesValidated --> EllipsoidEntry : "rejectLatRange [latitude outside -90..90] / showRangeWarning"
    CoordinatesValidated --> EllipsoidEntry : "rejectLonRange [longitude outside -180..180] / showRangeWarning"
    CoordinatesValidated --> CoordinateSystemChoice : "rejectMismatch [coordinate type != active case] / showMismatchError"
    CoordinatesValidated --> EllipsoidEntry : "rejectPrecision [frac-digits exceeded] / showPrecisionError"
    CoordinatesValidated --> CartesianEntry : "rejectPrecision [frac-digits exceeded] / showPrecisionError"
    ReferenceFrameEntry --> ReferenceFrameEntry : "rejectBodyFormat [astronomicalBody illegal chars] / showCharError"
    ReferenceFrameEntry --> ReferenceFrameEntry : "rejectDatumFormat [geodeticDatum illegal chars] / showCharError"
    ReferenceFrameEntry --> ReferenceFrameEntry : "rejectAltSystemNoBody [alternateSystem set, no body] / showMissingBodyError"
    ReferenceFrameResolved --> ReferenceFrameResolved : "warnAccuracy [accuracy exceeds datum capability] / showAccuracyWarning"
    VelocityEntry --> TemporalEntry : "setVelocity [v-north v-east v-up dec64 fr12] / acceptOrSkip"
    TemporalEntry --> GeoLocationPersisted : "setTimestampAndValidity [timestamp, optional valid-until] / validateChronology"
    TemporalEntry --> TemporalEntry : "rejectExpiredValidity [validUntil before timestamp] / showChronologyError"
    VelocityEntry --> TemporalEntry : "skip [no velocity]"
    GeoLocationPersisted --> [*]
```

## 7. Operational Context
From RFC 9179 Section 2.1, Frame of Reference:

> The frame of reference ('reference-frame') defines what the location values refer to and their meaning. The referred-to object can be any astronomical body. It could be a planet such as Earth or Mars, a moon such as Enceladus, an asteroid such as Ceres, or even a comet such as 1P/Halley. This value is specified in 'astronomical-body' and is defined by the International Astronomical Union. The default 'astronomical-body' value is 'earth'.

From RFC 9179 Section 2.2, Location:

> This is the location on, or relative to, the astronomical object. It is specified using two or three coordinate values. These values are given either as 'latitude', 'longitude', and an optional 'height', or as Cartesian coordinates of 'x', 'y', and 'z'.

From RFC 9179 Section 2.3, Velocity:

> The 'velocity' container specifies the velocity of the located entity relative to the astronomical body. The velocity vector components are 'v-north', 'v-east', and 'v-up', all expressed in meters per second. If the entity is stationary, the velocity is zero.

From RFC 9179 Section 3, Geo-Location Object:

> The 'geo-location' grouping includes temporal attributes: a 'timestamp' recording when the location was determined, and an optional 'valid-until' timestamp indicating when the location information should be considered no longer valid. The timestamp uses the 'date-and-time' type defined in RFC 6991.

## 8. Realization Matrix
### Required User Stories
- [ ] #46 - [RFC9179-GEO Reference Frame Selection and Geodetic Datum Resolution](https://github.com/gintatkinson/3dgs-039/blob/main/docs/user-stories/us-15-rfc9179-geo-reference-frame-selection-and-geodetic-datum-resolution.md) (semantic linkage: this user story provides the resolution logic for astronomical-body defaulting to "earth", geodetic-datum lookup via IANA registry per astronomical body, accuracy override precedence, and parent reference-frame inheritance — exercised in Main Success Scenario steps 2-3 and Alternate Flows 5d, 5e, and 5g)
- [ ] #47 - [RFC9179-GEO Ellipsoid Coordinate Validation and Precision](https://github.com/gintatkinson/3dgs-039/blob/main/docs/user-stories/us-16-rfc9179-geo-ellipsoid-coordinate-validation-and-precision.md) (semantic linkage: this user story validates latitude/longitude/height decimal64 fraction-digit precision against the 16/16/6 fr-digit limits and ISO 6709:2008 coordinate range bounds — exercised in Main Success Scenario step 6 and Alternate Flows 5a, 5b, and 5h)
- [ ] #44 - [RFC9179-GEO Location Coordinate System Choice Resolution](https://github.com/gintatkinson/3dgs-039/blob/main/docs/user-stories/us-17-rfc9179-geo-location-coordinate-system-choice-resolution.md) (semantic linkage: this user story enforces exclusive choice between ellipsoid and Cartesian coordinate systems, handles case switching with value preservation, and governs coordinate transformation availability — exercised in Main Success Scenario steps 4-5 and Alternate Flow 5c)

### Required Features
- [ ] #37 - [RFC9179-GEO Reference Frame Definition](https://github.com/gintatkinson/3dgs-039/blob/main/docs/features/feat-14-rfc9179-geo-reference-frame-definition.md) (semantic linkage: defines the reference-frame container with astronomical-body default "earth", alternate-system conditional leaf, geodetic-system sub-container with geodetic-datum "wgs-84" default, coord-accuracy dec64 fr6, and height-accuracy dec64 fr6 — all leaves exercised in Main Success Scenario steps 2-3 and Alternate Flows 5d, 5e, and 5g)
- [ ] #38 - [RFC9179-GEO Ellipsoid Coordinate Location](https://github.com/gintatkinson/3dgs-039/blob/main/docs/features/feat-15-rfc9179-geo-ellipsoid-coordinate-location.md) (semantic linkage: defines the ellipsoid case of the location choice with latitude dec64 fr16, longitude dec64 fr16, and height dec64 fr6 — exercised in Main Success Scenario steps 5-6 and Alternate Flows 5a, 5b, and 5h)

## Source References
Structural Schema: [ietf-geo-location@2022-02-11.yang](https://raw.githubusercontent.com/gintatkinson/3dgs-039/main/schema/ietf-geo-location%402022-02-11.yang) (Clause: grouping geo-location, containers reference-frame, location with choice ellipsoid/cartesian, velocity, leaves timestamp and valid-until)
Normative Specification: [RFC 9179: A YANG Grouping for Geographic Locations](https://datatracker.ietf.org/doc/rfc9179/) (Clause: Sections 2.1 Frame of Reference, 2.2 Location, 2.3 Velocity, 3 Geo-Location Object)
