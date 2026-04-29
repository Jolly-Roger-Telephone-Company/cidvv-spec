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
* CIDVV Platform: A system implementing this protocol

# Protocol Overview

CIDVV uses special Caller-ID prefixes to signal protocol operations:

* "10" prefix — Vouching
* "11" prefix — Vetting

CIDVV exchanges occur using short signaling dialogs and do not require media establishment.

# Protocol Operation

## Vouching Procedure

1. When Alice initiates a call to Bob, Alice's CIDVV platform MUST cache the tuple (Calling Number, Called Number), retain this cache entry for approximately 10 seconds, and reject the call attempt with SIP response 486 (Busy Here).

2. Alice's call MUST then proceed normally through the PSTN.

3. Upon receiving the call, Bob's system MUST initiate a verification call to Alice, prefix the Calling Party Number with "10", and use the originally dialed number as the destination.

4. Upon receiving a call with a "10" prefix, Alice's CIDVV platform MUST strip the "10" prefix, swap the calling and called numbers, and search for a matching cache entry.

5. If a match is found, the platform MUST respond with 486 (Busy Here).

6. If no match is found, the platform MUST respond with 603 (Decline).

7. Bob's system MUST treat a 486 response as a successful vouch and MUST treat any other response as a failed vouch.

## Vetting Procedure

1. Alice and Bob MUST agree on a shared secret, a calling number, and a validity time window.

2. Alice initiates a call using a Caller-ID prefixed with "11".

3. Upon receiving the call, Bob's CIDVV platform MUST strip the "11" prefix, compute SHA256(called-number || shared-secret), convert the result to a decimal representation, extract a fixed-length numeric value, cache this value for a short duration, and respond with 404 (Not Found).

4. Alice MUST perform the same computation and initiate a second call using the computed value, with "11" prefix, as Caller-ID.

5. Upon receiving the second call, Bob's CIDVV platform MUST verify that the value matches a cached entry and respond with 486 (Busy Here) if valid.

6. Alice MUST treat receipt of 486 as a successful vet.

7. Any deviation from this sequence MUST be treated as failure.
8. 
# Examples

## Vouching Call Flow

    Alice (CIDVV)        PSTN            Bob (CIDVV)
         |                 |                  |
         | INVITE          |                  |
         |---------------> |                  |
         | 486             |                  |
         |<--------------- |                  |
         |                 | INVITE           |
         |                 |--------------->  |
         |                 |                  |
         |<------INVITE----|                  |  (CallerID: 10...)
         |------486------->|                  |
         |                 |                  |
         |                 |  (Vouch success) |

## Vetting Call Flow

    Alice               PSTN             Bob
      |                   |                |
      | INVITE (11...)    |                |
      |------------------>|                |
      |                   |--------------->|
      |                   | 404            |
      |<------------------|<---------------|
      |                   |                |
      | INVITE (11+hash)  |                |
      |------------------>|                |
      |                   |--------------->|
      |                   | 486            |
      |<------------------|<---------------|

# Security Considerations

* Replay attacks are limited by short cache duration.
* Implementations MUST rate-limit CIDVV signaling traffic.
* CIDVV relies on correctness of PSTN routing and LERG data.
* Shared secrets used in vetting MUST be protected.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

The authors thank contributors to telephony security research and PSTN infrastructure development.
