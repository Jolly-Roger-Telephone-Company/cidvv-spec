---
title: CallerID Vouching and Vetting (CIDVV)
abbrev: CIDVV
category: info
docname: draft-anderson-askew-cidvv-latest
submissiontype: IETF
ipr: trust200902
date: 2026-04
consensus: false
v: 3
keyword: telephony, callerid, spoofing, PSTN, LERG

venue:
  github: "jollyrogertelephone/draft-cidvv"
  latest: "https://jollyrogertelephone.github.io/draft-cidvv/draft-anderson-askew-cidvv.html"

author:
  -
    ins: R. Anderson
    name: Roger Anderson
    organization: Jolly Roger Telephone Company
    email: roger@jollyrogertelephone.com
    country: US
  -
    ins: P. Askew
    name: Phillip Askew
    country: US

--- abstract

This document describes CallerID Vouching and Vetting (CIDVV), a lightweight protocol that improves trust in Caller-ID using existing Public Switched Telephone Network (PSTN) routing and numbering constraints.

CIDVV defines two mechanisms:

* Vouching — verification that a call originates from the authorized service provider for a given telephone number.
* Vetting — verification that an entity controls a telephone number.

CIDVV requires no centralized databases, certificates, or new signaling protocols and operates using existing PSTN infrastructure.

--- middle

# Introduction

Caller-ID spoofing is a global problem affecting all users of the telephone network. Existing mechanisms do not reliably bind signaling identity to authoritative numbering resources.

CIDVV addresses this problem by leveraging:

1. The Local Exchange Routing Guide (LERG), which defines authoritative routing ownership for telephone numbers.
2. Reserved numbering space within E.164, specifically prefixes "10" and "11", which are invalid in the North American Numbering Plan and can therefore be used as signaling indicators.

CIDVV operates entirely within normal PSTN routing behavior and does not require media exchange.

# Terminology

* Alice: Calling party
* Bob: Called party
* Mallory: Attacker attempting Caller-ID spoofing
* OSP: Originating Service Provider
* TSP: Terminating Service Provider
* LERG: Local Exchange Routing Guide

# Protocol Overview

CIDVV uses special Caller-ID prefixes to signal protocol operations:

* "10" prefix — Vouching
* "11" prefix — Vetting

CIDVV exchanges occur using short signaling dialogs and do not require media establishment.

# CallerID Vouching

## Overview

Vouching verifies that the originating service provider is authorized to originate calls for a telephone number.

## Successful Vouch

1. Alice calls Bob.
2. Alice's CIDVV platform caches the attempted call (Caller → Callee) for a short duration (~10 seconds) and returns 486 (Busy).
3. The call proceeds normally through the PSTN to Bob.
4. Bob's system initiates a verification call using a "10" prefixed Caller-ID.
5. Alice's CIDVV platform:
   * Recognizes the "10" prefix
   * Matches the call against its cache
   * Returns 486 (Busy) if a match is found
6. Bob treats this as a successful vouch and completes the call.

## Failed Vouch

If no cache match exists, Alice's CIDVV platform returns 603 (Decline). Bob treats the call as untrusted.

## Properties

* No central authority required
* Uses existing PSTN routing
* Prevents spoofing without access to authorized infrastructure
* Works across SIP, SS7, and TDM networks

# CallerID Vetting

## Overview

Vetting proves control of a telephone number using a two-call challenge-response exchange.

## Successful Vet

1. Alice and Bob agree on:
   * Shared secret
   * Vetting Caller-ID
   * Time window
2. Alice initiates a call using a Caller-ID prefixed with "11".
3. Bob's CIDVV platform:
   * Computes a SHA256-based challenge value
   * Caches the result
   * Returns 404 (Not Found)
4. Alice computes the same value and places a second call using the computed value as Caller-ID (with "11" prefix).
5. Bob verifies the value and returns 486 (Busy).
6. Alice records a successful vet.

## Failure Conditions

Vetting fails if:

* CIDVV is not implemented on the destination
* Shared parameters do not match
* Time window expires

# Security Considerations

* Replay attacks are limited by short cache duration.
* Implementations MUST rate-limit CIDVV signaling.
* CIDVV relies on trust in PSTN routing and LERG data.
* Vetting requires secure handling of shared secrets.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

The authors thank contributors to telephony security research and PSTN infrastructure development.
