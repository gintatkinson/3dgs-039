---
title: "[RFC9179-GEO] Reference Frame Selection and Geodetic Datum Resolution"
type: "user-story"
generation_mode: "subagent"
spec_source: "RFC 9179 Section 2.1 (ietf-geo-location@2022-02-11.yang, reference-frame container)"
issue_id: 46
---

# User Story: [RFC9179-GEO] Reference Frame Selection and Geodetic Datum Resolution

## Parent Epic
- [ ] #42 - [RFC9179-GEO] A YANG Grouping for Geographic Locations](https://github.com/gintatkinson/3dgs-039/blob/main/docs/epics/epic-04-rfc9179-geo-location.md) (semantic linkage: this user story exercises the reference-frame container resolution logic — default astronomical-body, geodetic-datum lookup by body, accuracy override precedence, and nested inheritance — as defined by RFC 9179 Section 2.1)

## Domain Object Mapping
- **Primary Domain Objects:** GeoLocation, ReferenceFrame, GeodeticSystem
- **Actor/Role:** GeoLocationConfigurator — the operator or automated subsystem that supplies raw or partial reference-frame data and requires the system to resolve defaults, select the appropriate geodetic datum, and inherit a parent location's reference frame when none is explicitly supplied

## BDD Scenario (OOA/OOD Realization)
**As a** GeoLocationConfigurator
**I want to** resolve the applicable reference frame and geodetic datum for a geo-location when only partial or no reference-frame data is supplied
**So that** every location always has a fully resolved astronomical body, geodetic datum, and accuracy context consistent with the schema defaults and parent-location inheritance rules

**Given** a GeoLocation resolver configured with the RFC 9179 Section 2.1 reference-frame schema defaults and the IANA Geodetic System Values Registry
**When** a geo-location is submitted with an unspecified astronomical-body, an unspecified geodetic-datum, and no parent reference-frame to inherit
**Then** the resolver applies the default astronomical-body "earth", selects the default geodetic-datum "wgs-84", and preserves any explicit coord-accuracy or height-accuracy values that override the datum defaults

## UML Sequence Diagram
```mermaid
sequenceDiagram
    autonumber
    actor configurator as "configurator : GeoLocationConfigurator"
    participant geoLocation as "geoLocation : GeoLocation"
    participant referenceFrame as "referenceFrame : ReferenceFrame"
    participant geodeticSystem as "geodeticSystem : GeodeticSystem"

    configurator->>geoLocation: resolveReferenceFrame()
    alt [referenceFrame is absent and parentFrame is available]
        Note over geoLocation, referenceFrame: Inherit parent reference-frame — copy all leaf values into local frame
        referenceFrame->>referenceFrame: setAstronomicalBody(inheritedBody)
        referenceFrame->>geodeticSystem: setGeodeticDatum(inheritedDatum)
    else [referenceFrame is absent and no parent exists]
        Note over geoLocation: Apply schema defaults for missing frame
        geoLocation->>referenceFrame: setAstronomicalBody("earth")
        referenceFrame->>referenceFrame: resolveGeodeticDatum("earth")
        referenceFrame-->geodeticSystem: resolvedDatum = "wgs-84"
        referenceFrame->>geodeticSystem: setGeodeticDatum("wgs-84")
    else [referenceFrame is explicit]
        alt [astronomicalBody is unspecified]
            geoLocation->>referenceFrame: setAstronomicalBody("earth")
        else [astronomicalBody is specified]
            Note over referenceFrame: Use explicit body value — validate ASCII pattern [ -@[-^_-~]*
        end
        referenceFrame->>referenceFrame: resolveGeodeticDatum(astronomicalBody)
        alt [astronomicalBody == "earth"]
            referenceFrame-->geodeticSystem: resolvedDatum = "wgs-84"
        else [astronomicalBody == "moon"]
            referenceFrame-->geodeticSystem: resolvedDatum = "me"
        else [astronomicalBody is other IAU body]
            Note over referenceFrame: Lookup IANA Geodetic System Values Registry
            referenceFrame-->geodeticSystem: resolvedDatum = ianaRegistryValue
        end
        alt [geodeticDatum is unspecified]
            referenceFrame->>geodeticSystem: setGeodeticDatum(resolvedDatum)
        else [geodeticDatum is explicit]
            Note over geodeticSystem: Validate explicit value — apply uppercase-to-lowercase conversion
            geodeticSystem->>geodeticSystem: setGeodeticDatum(explicitValue)
        end
    end
    alt [coordAccuracy is specified]
        Note over geodeticSystem: Override datum default horizontal accuracy with decimal64 6-fraction-digit value
        referenceFrame->>geodeticSystem: setCoordAccuracy(coordAccuracy)
    else [coordAccuracy is unspecified]
        Note over geodeticSystem: Retain datum default accuracy
    end
    alt [heightAccuracy is specified]
        Note over geodeticSystem: Override datum default height accuracy — applies to ellipsoidal coordinates only
        referenceFrame->>geodeticSystem: setHeightAccuracy(heightAccuracy)
    else [heightAccuracy is unspecified]
        Note over geodeticSystem: Retain datum default height accuracy
    end
    referenceFrame-->geoLocation: resolvedFrame : ReferenceFrame
    alt [alternateSystems feature is active and alternateSystem is set]
        Note over referenceFrame: Apply alternate-system — modifies definitions but not types of other reference-frame values
    else [alternateSystems feature is inactive]
        Note over referenceFrame: Natural universe context — no modification
    end
    geoLocation-->configurator: resolutionResult = FullReferenceFrame
```

## UML State Machine Diagram
```mermaid
stateDiagram-v2
    [*] --> FrameUnresolved : "createGeoLocation [no referenceFrame provided]"
    FrameUnresolved --> InheritingParent : "inherit [parentGeoLocation has referenceFrame] / copyParentFrame"
    FrameUnresolved --> ApplyingSchemaDefaults : "resolve [no parent available] / beginDefaultResolution"
    ApplyingSchemaDefaults --> BodyResolved : "setBody [astronomicalBody is absent] / applyDefault earth"
    ApplyingSchemaDefaults --> BodyExplicit : "setBody [astronomicalBody is present] / useExplicitValue"
    BodyResolved --> DatumLookupByBody : "resolveDatum [geodeticDatum is absent] / queryBodyDefault"
    BodyExplicit --> DatumLookupByBody : "resolveDatum [geodeticDatum is absent] / queryBodyDefault"
    DatumLookupByBody --> DatumResolved : "resolve [body == earth] / setDatum wgs-84"
    DatumLookupByBody --> DatumResolved : "resolve [body == moon] / setDatum me"
    DatumLookupByBody --> DatumResolved : "resolve [body is other IAU body] / setDatum IanaRegistryValue"
    BodyResolved --> DatumExplicit : "resolveDatum [geodeticDatum is present] / validateCharacters"
    BodyExplicit --> DatumExplicit : "resolveDatum [geodeticDatum is present] / validateCharacters"
    InheritingParent --> FrameResolved : "completeInheritance"
    DatumExplicit --> ApplyingAccuracyOverride : "validateDatum [ASCII pattern matches] / proceedToAccuracy"
    DatumResolved --> ApplyingAccuracyOverride : "resolveAccuracy [coord or height accuracy is present] / overrideDatumDefaults"
    DatumResolved --> FrameResolved : "resolveAccuracy [no accuracy overrides] / finalizeFrame"
    ApplyingAccuracyOverride --> FrameResolved : "apply [coordAccuracy or heightAccuracy specified] / setPrecision decimal64 6 fraction-digits"
    FrameResolved --> [*]
```

## Operational Context
From RFC 9179 Section 2.1:

> The frame of reference (`reference-frame`) defines what the location values refer to and their meaning. The referred-to object can be any astronomical body. It could be a planet such as Earth or Mars, a moon such as Enceladus, an asteroid such as Ceres, or even a comet such as 1P/Halley. This value is specified in `astronomical-body` and is defined by the International Astronomical Union. The default `astronomical-body` value is `earth`.

From the schema (ietf-geo-location@2022-02-11.yang) — Reference frame resolution semantics:
- **astronomical-body** defaults to `"earth"`; uppercase SHOULD be converted to lowercase; values match ASCII printable pattern `[ -@\[-\^_-~]*` (ranges 32-64, 91-126)
- **geodetic-datum** defaults to `"wgs-84"` when the astronomical body is `"earth"`; the IANA registry defines body-specific datum values (e.g., `"me"` for moon — Mean Earth/Polar Axis); values match the same ASCII printable pattern and IANA further restricts by converting spaces to dashes
- **coord-accuracy** and **height-accuracy** override the default accuracy implied by the geodetic-datum when specified; both are `decimal64` with 6 fraction-digits; height-accuracy carries `units "meters"` and applies to ellipsoidal coordinates only
- **alternate-system** is conditionally present (guarded by `if-feature "alternate-systems"`); when present it "modifies the definition (but not the type) of the other values in the reference frame"
- Nested location containers inherit the parent's reference-frame when their own reference-frame is absent (structural containment implies semantic inheritance)

## Required Features Matrix
- [ ] #37 - [RFC9179-GEO Reference Frame Definition](https://github.com/gintatkinson/3dgs-039/blob/main/docs/features/feat-14-rfc9179-geo-reference-frame-definition.md) (semantic linkage: this user story exercises the resolution, defaulting, and inheritance logic for every leaf and container defined in feat-14 — astronomical-body defaulting, geodetic-datum body-specific default resolution, coord-accuracy and height-accuracy override semantics, alternate-system conditional guarding, and the ReferenceFrame-to-GeodeticSystem containment path)

## Logical UI & Interface Bindings
- **Target LUI Component:** PropertyGrid (reference-frame section with conditional alternate-system visibility and nested geodetic-system subsection)
- **Target Layout Container ID:** properties_view
- **Data Source Bindings:** Deferred to Feature #37 Task 2

## Source References
Structural Schema: [ietf-geo-location@2022-02-11.yang](https://raw.githubusercontent.com/gintatkinson/3dgs-039/main/schema/ietf-geo-location%402022-02-11.yang) (Clause: reference-frame container, astronomical-body leaf with default "earth", alternate-system leaf with if-feature, geodetic-system container with geodetic-datum, coord-accuracy decimal64 fraction-digits 6, height-accuracy decimal64 fraction-digits 6 with units meters)
Normative Specification: [RFC 9179: A YANG Grouping for Geographic Locations](https://datatracker.ietf.org/doc/rfc9179/) (Clause: Section 2.1, Frame of Reference)
