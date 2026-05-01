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

Alice's CIDVV platform receives an attempted call from Alice to Bob. It MUST cache the calling number and called number for a short interval, normally about 10 seconds. It then rejects the attempt with SIP response 486 (Busy Here).

Alice's SBC then sends the call normally through the PSTN toward Bob.

When Bob's system receives the call, Bob's system initiates a verification call toward Alice. The verification call uses a Caller-ID formed by prefixing Bob's number with the digits "10".

When Alice's CIDVV platform receives a call with a Caller-ID beginning with "10", it MUST remove the "10" prefix, compare the resulting number pair against the recent cache, and determine whether the original call attempt exists.

If a matching cache entry exists, Alice's CIDVV platform MUST reject the verification call with SIP response 486 (Busy Here). Bob's system treats this response as a successful vouch.

If no matching cache entry exists, Alice's CIDVV platform MUST reject the verification call with SIP response 603 (Decline). Bob's system treats this response as a failed vouch.

## Vetting Procedure

Before vetting begins, Alice and Bob agree on a shared secret, Alice's vetting Caller-ID, and a validity time window.

Alice places a vetting call to Bob using a Caller-ID beginning with the digits "11".

When Bob's CIDVV platform receives the first vetting call, it removes the "11" prefix and verifies that the resulting Caller-ID is expected for the current vetting attempt. Bob's platform then computes a SHA-256 value over the called number followed by the shared secret. Bob's platform converts that value to decimal form, extracts a fixed-length numeric code, stores the code briefly, and rejects the call with SIP response 404 (Not Found).

Alice performs the same SHA-256 calculation and places a second vetting call to Bob. This second call uses a Caller-ID beginning with "11" followed by the computed numeric code.

When Bob's CIDVV platform receives the second vetting call, it removes the "11" prefix and compares the remaining numeric code to the recently cached value.

If the numeric code matches, Bob's CIDVV platform MUST reject the call with SIP response 486 (Busy Here). Alice's platform treats this response as a successful vet.

Any other response, timeout, code mismatch, expired cache entry, or unexpected Caller-ID MUST be treated as a failed vet.
# Examples

## Successful Vouch Call Flow
~~~~
   Alice      CIDVV_A      SBC_A        PSTN       SBC_B        Bob
     |           |           |           |           |           |
     |------- INVITE ------->|           |           |           |
     |  Step 1   |           |           |           |           |
     |           |           |           |           |           |
     |           |<- INVITE -|           |           |           |
     |           |  Step 2   |           |           |           |
     |           |           |           |           |           |
     |           |-- Busy -->|           |           |           |
     |           |  Step 3   |           |           |           |
     |           |           |           |           |           |
     |           |           |- INVITE ->|           |           |
     |           |           |  Step 4   |           |           |
     |           |           |           |           |           |
     |           |           |           |- INVITE ->|           |
     |           |           |           |  Step 5   |           |
     |           |           |           |           |           |
     |           |           |           |<- INVITE -|           |
     |           |           |           |  Step 6   |           |
     |           |           |           |           |           |
     |           |           |<- INVITE -|           |           |
     |           |           |  Step 7   |           |           |
     |           |           |           |           |           |
     |           |<- INVITE -|           |           |           |
     |           |  Step 8   |           |           |           |
     |           |           |           |           |           |
     |           |-- Busy -->|           |           |           |
     |           |  Step 9   |           |           |           |
     |           |           |           |           |           |
     |           |           |-- Busy -->|           |           |
     |           |           |  Step 10  |           |           |
     |           |           |           |           |           |
     |           |           |           |-- Busy -->|           |
     |           |           |           |  Step 11  |           |
     |           |           |           |           |           |
     |           |           |           |           |- INVITE ->|
     |           |           |           |           |  Step 12  |
     |           |           |           |           |           |
~~~~
{: #fig-successful-vouch title="Example Successful Vouch"}
### Successful Vouch Call Flow

The diagram above shows the high-level message flow. The following numbered steps provide the detailed behavior, including Caller-ID manipulation performed by the CIDVV platforms.

1. The originating user (Alice, Caller-ID `+12125550100`) initiates a call to the destination user (Bob, dialed number `+19495550199`).

2. The call is routed from Alice's User Agent to her SBC, which forwards it to the originating CIDVV platform (CIDVV_A).

3. **CIDVV_A**:
   - Caches the call attempt `(From: +12125550100, To: +19495550199)` for a short period (≈10 seconds).
   - Rejects the call with **486 Busy Here**.

4. Alice's SBC receives the 486 and advances the original call toward the PSTN using the original Caller-ID.

5. The call reaches Bob's SBC via the PSTN.

6. **Bob's SBC**:
   - Prefixes `10` to the dialed number, resulting in `1019495550199`.
   - Forwards the call back toward Alice's number (`+12125550100`) with the modified Caller-ID `+1019495550199`.

7. The return call arrives at Alice's SBC via the PSTN.

8. **Alice's SBC**:
   - Detects the leading `10` prefix on the Caller-ID.
   - Routes the call to CIDVV_A for vouch verification.

9. **CIDVV_A**:
   - Receives the call with `To: +12125550100` and `From: +1019495550199`.
   - Strips the `10` prefix from the From number, yielding `+19495550199`.
   - Swaps the From and To values.
   - Looks up the resulting pair `(+12125550100, +19495550199)` in its short-term cache.
   - Finds a matching entry from step 3.
   - Considers this a successful vouch and returns **486 Busy Here**.

10. Bob's SBC receives the 486 via the PSTN, recognizes it as a successful vouch response, and advances the original call to Bob's User Agent.

11. Bob's telephone rings.

This two-call mechanism (first vetting call + return vouch call) allows the originating CIDVV platform to confirm that the asserted Caller-ID is valid without completing the initial call.

## Failed Vouch Call Flow
The following diagram shows a failed vouch attempt by an impersonator (Mallory) who spoofs Alice's Caller-ID.
~~~~
  Mallory     CIDVV_A      SBC_A        PSTN       SBC_B    Voicemail_B
     |           |           |           |           |           |
     |------------- INVITE ------------->|           |           |
     |  Step 1   |           |           |           |           |
     |           |           |           |           |           |
     |           |           |           |- INVITE ->|           |
     |           |           |           |  Step 2   |           |
     |           |           |           |           |           |
     |           |           |           |<- INVITE -|           |
     |           |           |           |  Step 3   |           |
     |           |           |           |           |           |
     |           |           |<- INVITE -|           |           |
     |           |           |  Step 4   |           |           |
     |           |           |           |           |           |
     |           |<- INVITE -|           |           |           |
     |           |  Step 5   |           |           |           |
     |           |           |           |           |           |
     |           |-NotFound->|           |           |           |
     |           |  Step 6   |           |           |           |
     |           |           |           |           |           |
     |           |           |-NotFound->|           |           |
     |           |           |  Step 7   |           |           |
     |           |           |           |           |           |
     |           |           |           |-NotFound->|           |
     |           |           |           |  Step 8   |           |
     |           |           |           |           |           |
     |           |           |           |           |--- VM --->|
     |           |           |           |           |  Step 9   |
     |           |           |           |           |           |
~~~~
{: #fig-failed-vouch title="Example Failed Vouch"}
#### Step-by-step description

1. Mallory spoofs Alice’s Caller-ID (`+12125550100`) and initiates a call to Bob (`+19495550199`).

2. The call arrives at Bob’s SBC via the PSTN.

3. **Bob’s SBC**:
   - Prefixes `10` to the dialed number, creating `1019495550199`.
   - Forwards the call toward Alice’s number (`+12125550100`) with the modified Caller-ID `+1019495550199`.

4. The call arrives at Alice’s SBC via the PSTN.

5. **Alice’s SBC**:
   - Detects the `10` prefix and recognizes this as a vouch call.
   - Routes the call to CIDVV_A for verification.

6. **CIDVV_A**:
   - Processes `To: +12125550100` and `From: +1019495550199`.
   - Strips the `10` prefix, yielding `+19495550199`.
   - Swaps From and To values.
   - Looks up the pair `(+12125550100, +19495550199)` in its short-term cache.
   - Finds no matching entry (because Alice never placed the original call).
   - Rejects the call with **404 Not Found**.

7. The 404 propagates back through the PSTN to Bob’s SBC.

8. **Bob’s SBC** recognizes the 404 as a failed vouch and either rejects the call or forwards it to Bob’s voicemail. Bob remains unaware of the impersonation attempt.

This mechanism ensures that only calls that originated from a legitimate CIDVV platform (i.e., those that previously cached the attempt) will pass vouching. Spoofed or unsolicited calls are rejected early.

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
