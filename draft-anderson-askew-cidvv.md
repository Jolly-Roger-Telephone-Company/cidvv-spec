---
title: CallerID Vouching and Vetting (CIDVV)
abbrev: CIDVV
category: info
docname: draft-anderson-askew-cidvv
submissiontype: IETF
ipr: trust200902
date: 2026-04
consensus: false
v: 3
keyword: telephony, callerid, vouching, vetting, spoofing, LERG, PSTN

venue:
  github: "jollyrogertelephone/draft-cidvv"
  latest: "https://jollyrogertelephone.github.io/draft-cidvv/draft-anderson-askew-cidvv.html"

author:
  -
    ins: R. Anderson
    name: Roger Anderson
    organization: Jolly Roger Telephone Company
    email: roger@jollyrogertelephone.com
  -
    ins: P. Askew
    name: Phillip Askew
--- abstract

This document describes CIDVV ("CallerID Vouching and Vetting"), a lightweight protocol that uses the existing Public Switched Telephone Network (PSTN) and the North American Local Exchange Routing Guide (LERG) to provide strong cryptographic-style trust signals for Caller-ID without requiring central databases, certificates, or new signaling protocols such as STIR/SHAKEN.

CIDVV defines two complementary mechanisms:

* **Vouching** — proves that the originating carrier is authorized (per LERG) to send calls from a given number.
* **Vetting** — allows a telephone number owner to prove ownership/control to third parties (branding services, trade organizations, etc.) using only PSTN signaling.

Both mechanisms exploit otherwise-invalid E.164 prefixes "10" and "11" to create a side-channel signaling path carried entirely within normal PSTN routing.

--- middle

# Introduction

Virtually every telephone user worldwide receives spoofed, spam, and scam calls. Caller-ID has become almost meaningless because the signaling path and the authoritative routing tables (LERG) are only loosely coupled. This document defines a simple, deployable mechanism that closes that gap.

CIDVV leverages two key facts:

1. The LERG provides authoritative mapping of every NPA-NXX to its authorized service provider.
2. The North American Numbering Plan reserves leading digits "10" and "11" after the country code as invalid for normal E.164 numbers, creating safe prefix space for signaling.

By prefixing called or calling numbers with these digits and using brief rejected call dialogs, CIDVV achieves strong vouching and vetting with zero billable media and no central authority.

The domain `cidvv.org` has been procured for future reference material, code, and community support.

# Terminology

* **Alice**: The calling party (originator) who presents a telephone number as Caller-ID.
* **Bob**: The called party (terminator) who receives the call and relies on the displayed/vetted Caller-ID.
* **Eve**: A passive eavesdropper who can observe signaling or media but does not alter calls.
* **Mallory**: A malicious active attacker (scammer, robocaller, or spoofing operator) who forges Caller-ID or attempts to inject deceptive calls.
* **OSP (Originating Service Provider)**: The carrier authorized by LERG to originate calls from a given number; issues vouches.
* **TSP (Terminating Service Provider)**: The carrier delivering the call to Bob; performs vetting.
* **LERG**: Local Exchange Routing Guide — the authoritative monthly database maintained by iconectiv that maps every NPA-NXX to its authorized Operating Company Number (OCN) and interconnection points.
* **Vouch**: A successful challenge-response exchange proving that the OSP is authorized for the claimed Caller-ID.
* **Vet**: A successful two-call exchange proving ownership/control of a telephone number to a third party.

# Vouching Mechanism

## Successful Vouch Example

1. Alice (number 1-213-555-0175) wants to call Bob (1-916-555-0128). Both have CIDVV-capable platforms.
2. Alice's SBC sends the call to her CIDVV platform.
3. The CIDVV platform caches the attempt (from 12135550175 → 19165550128) for ~10 seconds and returns 486 User Busy.
4. Alice's SBC routes the call normally into the PSTN.
5. The call arrives at Bob's SBC.
6. Bob's SBC prefixes the dialed number with "10" (1019165550128) and routes a new call to Alice's number using the prefixed number as Caller-ID.
7. Alice's SBC recognizes the "10" prefix, forwards to her CIDVV platform.
8. The CIDVV platform strips the "10", swaps From/To, finds the matching cache entry, and returns 486 User Busy.
9. Bob's SBC receives the 486, recognizes a successful vouch, and delivers the original call to Bob.

# Vetting Mechanism

## Successful Vetting Example

1. Alice wants to vet Bob's number (1-916-555-0128). They share a secret key ("hamburger") and agree on Alice's vetting Caller-ID (1-213-555-0175) and a time window.
2. Alice's CIDVV platform calls Bob using prefixed Caller-ID 1112135550175.
3. Bob's CIDVV platform recognizes the "11" prefix, strips it, computes SHA256("19165550128" + "hamburger"), takes the first 10 digits of the decimal result (e.g., 1918486666), caches it, and returns 404 Not Found.
4. Alice's platform repeats the calculation and calls again using 111918486666.
5. Bob's platform recognizes the matching cached code and returns 486 User Busy.
6. Alice's platform reports success to Alice.

# Security Considerations

* Cache timing attacks — The short cache window (≈10 seconds) limits replay windows.
* Denial of service — Implementations MUST rate-limit CIDVV challenge calls.
* Billing implications — All CIDVV exchanges are short SIP dialogs that should not generate billable minutes.
* International routing — Behavior of "10" and "11" prefixes outside NANP must be verified per destination country.
* Privacy — Vetting exchanges reveal only that a number owner is participating; no media is exchanged.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

TODO
