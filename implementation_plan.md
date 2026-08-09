# Implementation Plan: Specification-Engineer RFC 6991 ietf-inet-types (2013-07-15)

## Summary

Specification-engineer the **ietf-inet-types@2013-07-15** YANG module (RFC 6991 revision) using the DEAP spec-orchestrator pipeline. This is the Internet Protocol Suite types module with ~14 typedefs from the RFC 6991 era.

**Existing workspace context:** ietf-inet-types@2025-12-22 (RFC 9911) is already spec'd. The 2013 revision is a subset. All new specs will use `[RFC6991-INET]` namespace to avoid collision.

## Phase 0: Pre-flight

- Schema already exists at `schema/ietf-inet-types@2013-07-15.yang`? (Check; download if missing)
- Bootstrap tracker labels (already done)
- Run YANG-to-LUI compiler if applicable

## Phase 1: Structural Extraction (Worker A)

1 Epic: `[RFC6991-INET] Internet Protocol Suite Types`
~4 Features:
- Protocol Field Types (dscp, ipv6-flow-label, port-number, as-number)
- IP Address Types (ip-version, ip-address, ipv4-address, ipv6-address, ip-address-no-zone, ipv4-address-no-zone, ipv6-address-no-zone)
- IP Prefix Types (ip-prefix, ipv4-prefix, ipv6-prefix)
- Domain, Host, and URI Types (domain-name, host, uri)

## Phase 2: Behavioral Extraction (Worker B)

~4-6 User Stories covering: IP address formatting/validation, zone handling, prefix validation, domain-name validation, host union resolution, port number range checking, as-number format

## Phase 3: System Interaction Extraction (Worker C)

~2-3 Use Cases: validating IP addresses against type constraints, resolving host identifiers, managing prefix-based network addressing

## Phase 4-5: Verification, Reconciliation, Reporting

All standard gates. Title namespace: `[RFC6991-INET]`.
