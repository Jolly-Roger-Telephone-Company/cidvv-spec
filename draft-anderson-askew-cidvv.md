---
title: Caller-ID Vouching and Vetting (CIDVV)
abbrev: CIDVV
category: info
docname: draft-anderson-askew-cidvv-latest
submissiontype: IETF
ipr: trust200902
date: 2026-05-07
consensus: false
v: 3
keyword: telephony, callerid, spoofing, PSTN

venue:
  github: "Jolly-Roger-Telephone-Company/cidvv-spec"
  latest: "https://cidvv.org/draft-anderson-askew-cidvv.html"
author:
  -
    ins: R. Anderson
    name: Roger Anderson
    organization: Jolly Roger Telephone Company
    email: roger@jollyrogertelephone.com
    country: US
  -
    ins: S. Berkson
    name: Steven Berkson
    organization: Jolly Roger Telephone Company
    email: steveb@jollyrogertelephone.com
    country: US
  -
    ins: P. Askew
    name: Phillip Askew
    email: phillip.askew@theaskewcrew.com
    country: US
---

<!--
Headings (starting with #) must be separated by a blank line. Can use this regex to find: [ \t]$
-->

--- abstract

Caller-ID spoofing remains a significant problem in telephony, particularly across inter-domain and international call paths where identity frameworks may not yet be fully deployed.

This document defines **Caller-ID Vouching and Vetting (CIDVV)**, a lightweight verification mechanism that lets the called party ask a simple question:

> "Will the party responsible for this number vouch for this call right now?"

CIDVV uses short-lived signaling exchanges encoded within the Calling Party Number to confirm that the calling party controls the Asserted Caller-ID. It is designed to operate across heterogeneous SIP and SS7/TDM networks without requiring new protocol extensions or persistent identity infrastructure. It relies on existing call routing behavior and intentionally leverages failure responses as a signaling mechanism.

CIDVV is complementary to STIR/SHAKEN and other identity frameworks. It provides an incrementally deployable tool that works even in environments where cryptographic attestation is not yet available or sufficient, while remaining fully tolerant of intermediate network modification.

By requiring demonstrable real-time control of the Asserted Caller-ID, CIDVV strengthens resistance to spoofing in a practical, low-overhead manner.

--- middle

# Introduction

CIDVV supports two closely related functions: **Vouching** (real-time verification that the asserted number vouches for a specific call) and **Vetting** (confirmation that a number is controlled by its expected owner, useful for branding, trust programs, and registries). The primary focus of this document is the vouching mechanism, which directly addresses Caller-ID spoofing for individual calls.

Caller-ID spoofing remains a widespread problem in modern telephony. Fraudulent and nuisance callers frequently impersonate legitimate numbers, eroding trust and complicating call screening for recipients.

This document defines **Caller-ID Vouching and Vetting (CIDVV)**, a lightweight, incrementally deployable mechanism that allows the called party to ask a simple real-time question:

> "Will the party responsible for this number vouch for this call right now?"

CIDVV verifies caller identity through network reachability rather than relying solely on asserted identity. It requires that a party asserting a Caller-ID demonstrate control of that number by being able to receive a short return signaling call within a brief Validity Window.

CIDVV operates by encoding signaling information within the Calling Party Number and leveraging existing call routing behavior to perform a challenge-response exchange. The protocol requires no new SIP headers, protocol extensions, response codes, or changes to SS7 signaling. It is designed to function across mixed SIP and TDM networks, including international paths.

**CIDVV is complementary to STIR/SHAKEN** and other identity frameworks. It provides practical protection in environments where cryptographic attestation is not yet fully deployed, while remaining fully tolerant of signaling modifications by intermediate networks. It intentionally uses distinct failure-response behaviors as part of its signaling mechanism and does not require universal adoption to deliver benefit.

The mechanism leverages two key elements of the existing telephone ecosystem:

* Authoritative routing databases and numbering plans, which establish ownership of telephone numbers.
* Short digit sequences (e.g., "100" and "101") chosen to minimize conflicts with valid numbering plans.

CIDVV operates entirely within standard PSTN routing behavior and requires no media exchange. While it does not provide absolute identity assurance, it delivers strong, real-time evidence of Caller-ID control in a practical and low-overhead manner.

# Terminology

* **Caller-ID**: The telephone number presented to the called party (what the end user sees).
* **Asserted Caller-ID**: The Caller-ID value that is being vouched or vetted by this protocol. This is the number whose control the calling party claims, and it is the value used for the CIDVV Token, state management, and correlation.
* **Calling Party Number**: The value carried in the signaling protocol (e.g., SIP `From` header or ISUP Calling Party Number parameter). In many deployments this is the same as the presented Caller-ID, but they are not always identical.
* **Alice**: The calling party and verifier. In vouching flows Alice asserts a number; in vetting flows Alice verifies Bob's number.
* **Bob**: The called party. In vetting flows Bob is the owner whose number is being vetted.
* **Mallory**: An attacker attempting to spoof a Caller-ID.
* **CIDVV Platform**: A system that implements the vouching and vetting procedures defined in this document.
**CIDVV-aware Network Element**: A network element (typically an SBC or proxy) that recognizes CIDVV signaling prefixes ("100" and "101") in the Calling Party Number and routes those calls to a CIDVV platform. It is also responsible for forwarding initial INVITEs for new dialogs to the CIDVV platform and handling specific responses (such as 404) to advance the original call.
* **Vouch**: The act of a CIDVV platform asserting that it has verified control of a telephone number through the challenge-response mechanism described in this document, which may consist of one or more verification calls. A successful vouch provides strong evidence that the calling party controls the Asserted Caller-ID.
* **Vet** (or **Vetting**): The process by which a CIDVV platform confirms the relevant party controls the Asserted Caller-ID via the three-call challenge-response sequence. Vetting may be performed on behalf of third parties such as Caller-ID branding services, Vetting Agents, law enforcement agencies, trade organizations, or enterprise trust programs.
* **Vouching Call**: A short signaling call used in the CIDVV protocol. CIDVV defines **Phase 1** ("100" prefix) and **Phase 2** ("101" prefix) verification calls.
* **Phase 1 Vouch** ("100" prefix): The initial Vouch verification step. Expected response behavior is a "Busy"-class response (e.g., SIP 486 Busy Here).
* **Phase 2 Vouch** ("101" prefix): The secondary Vouch step. Expected response behavior is a "Rejection"-class response (e.g., SIP 603 Decline).
* **Successful Vouch**: Requires **both Phase 1 and Phase 2** to complete with the expected behaviors within the Validity Window.
* **Unsuccessful Vouch**: A verification result indicating that no matching cache entry was found.
* **Verification Not Performed**: A condition where verification could not be completed due to system or network conditions.
* **Validity Window**: The time interval during which the originating CIDVV platform will accept and correlate a vouch attempt (return call) from the called party. This is typically on the order of 10-30 seconds.
* **Vouch-Call Timeout**: The local timer used by the originating CIDVV platform to limit how long it will wait for a response to an individual Phase 1 or Phase 2 vouching call. This is typically 3-6 seconds for domestic calls and longer (e.g. 8-20 seconds) for international calls. It is distinct from the Validity Window.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14, RFC 2119 and RFC 8174 when, and only when, they appear in all
capitals, as shown here.

## Motivation and Advantages

While mechanisms such as STIR/SHAKEN provide important cryptographic caller identity assurance, their deployment is still partial, particularly across international and inter-provider boundaries. CIDVV is designed as a pragmatic, complementary approach that works today using only existing telephone infrastructure.

The primary advantages of CIDVV are:

* **Leverages existing PSTN infrastructure**: Requires no new protocol extensions, headers, or infrastructure. It uses established numbering plans and routing databases.

* **Strong reachability-based anti-spoofing**: A successful vouch gives strong evidence that the calling party controls the Asserted Caller-ID, because only the legitimate owner can respond to the short signaling challenge.

* **Visibility into spoofing**: Number owners and their providers gain valuable real-world telemetry on spoofing attempts against their numbers through logged vetting requests.

* **Low signaling overhead**: Short, media-less verification calls replace what would otherwise be completed fraudulent calls, reducing overall network load.

* **Broad compatibility**: Works natively across SIP, SS7/TDM, ISDN, and mixed networks, including international paths. No SIP-specific features are required.

* **Incremental and flexible deployment**: Can be implemented by enterprises (using tools such as Kamailio or Asterisk), service providers, or third-party services. No universal adoption is needed to provide benefit.

* **Promotes competition and innovation**: Third-party cloud-based vouching/vetting services allow easy provider switching, lowering costs and encouraging a healthy ecosystem.

By avoiding centralized authorities and single points of failure, CIDVV lowers the barrier to deployment while providing immediate, practical value in the fight against Caller-ID spoofing.

## Design Principles

CIDVV was designed with the following core principles:

* **Maximal compatibility with existing infrastructure**: The protocol must work across SIP, SS7/TDM, ISDN, and mixed networks - including international paths - without requiring changes to signaling protocols, new headers, response codes, or media support.

* **Resilience to intermediate network behavior**: Intermediate networks may normalize, truncate, or otherwise modify signaling information. Therefore, all protocol state is encoded in a compact numeric form within the Calling Party Number field, which has the highest chance of surviving end-to-end.

* **Use of existing failure semantics**: CIDVV relies on distinguishable classes of call rejection behavior (e.g., "busy" vs. "decline") rather than requiring specific end-to-end response codes. Failed call attempts are intentionally used as a lightweight signaling mechanism.

* **Minimal new infrastructure**: No persistent identity infrastructure, central authorities, or cryptographic key management is required. The mechanism leverages existing routing databases and numbering authority.

* **Incremental deployability**: The protocol provides benefit even with partial adoption and is designed to coexist cleanly with STIR/SHAKEN and other identity solutions.

* **Practical and low-overhead operation**: Verification uses very short signaling-only calls with no media exchange, keeping network impact minimal while still providing strong real-time evidence of number control.

These principles ensure CIDVV can be deployed quickly and broadly while delivering meaningful protection against Caller-ID spoofing today.

## Simple Overview
{: #simple-overview }

CIDVV defines two related operations:

* **Vouching** — Allows the called party (or their provider) to verify in real time whether the party responsible for the presented Caller-ID vouches for *this specific call*.

* **Vetting** — Allows confirmation that a telephone number is under the control of its expected owner. This is useful for Caller-ID branding services, enterprise trust programs, industry registries, and similar applications.

### Vouching Operation (Primary Use Case)

When Alice wants to place a call to Bob while asserting a particular Caller-ID:

1. Alice places a normal call to Bob using the Asserted Caller-ID. Alice's CIDVV platform is notified of the outbound call attempt.
2. Bob's CIDVV platform intercepts the incoming call before ringing Bob's phone.
3. Bob's platform initiates **two short signaling-only verification calls** back to Alice's Asserted Caller-ID (these may be performed in parallel):
   - **Phase 1** verification call using Calling Party Number prefix `"100"`.
   - **Phase 2** verification call using Calling Party Number prefix `"101"`.
4. Alice's CIDVV platform recognizes the special prefixes on the incoming verification calls and responds with the expected rejection behavior for each phase. This proves that it controls the Asserted Caller-ID **and** that Alice has an active call in progress to Bob.
5. If both Phase 1 and Phase 2 verifications succeed within the Validity Window, Bob's CIDVV platform allows the original call to ring through to Bob.

The two verification calls use **reachability testing** to confirm that Alice (or her service provider) genuinely controls the Asserted Caller-ID she is presenting for this specific call.

### Vetting Operation

Vetting allows a party (Alice) to confirm that another party (Bob) controls a specific telephone number. It is particularly useful for Caller-ID branding services, trust programs, and registries.

Vetting uses a three-call challenge-response sequence consisting of a **Wake Call**, **Recognize Call**, and **Auth Call**, all protected by a pre-shared secret. The sequence is designed to prevent an attacker from goading the number owner into placing return calls (e.g., by spoofing well-known vetting numbers).

When Alice wants to vet that Bob controls a particular telephone number:

1. Alice and Bob share a secret (e.g., a passphrase such as "elephant").
2. Alice initiates a **Wake Call** to Bob using a special prefix (`101`) and an agreed vetting Caller-ID. Bob's platform recognizes the vetting Caller-ID, computes a short-lived Recognize Token, and rejects the call with the standard response code **603**.
3. Alice's platform then sends a **Recognize Call** using the Recognize Token in the Calling Party Number.
4. Bob's platform verifies the token (proving Alice knows the shared secret) and responds with an **Auth Call** using an Auth Token.
5. Alice's platform verifies the Auth Token (proving Bob knows the shared secret). Both sides now consider the vetting successful.

This design ensures that the initial Wake Call reveals nothing useful to an attacker, while the subsequent Recognize and Auth steps provide mutual authentication between parties that share the secret. All calls remain short signaling-only exchanges with no media.

**Note**: The shared secret and token exchange details are defined in Section [Token Computation Algorithm](#token-computation).

## CIDVV Mechanisms

### Vouching Mechanism

CIDVV uses two distinct signaling prefixes in the Calling Party Number for vouching:

* **"100"** — Phase 1 Verification Call
* **"101"** — Phase 2 Verification Call

A successful **Vouch** requires **both** Phase 1 and Phase 2 to complete with their expected responses within the Validity Window. The two phases MAY be performed in any order or in parallel.

**Expected behaviors**:
* **Phase 1** ("100" prefix): MUST receive a Busy-class response (e.g., SIP 486 Busy Here).
* **Phase 2** ("101" prefix): MUST receive a Rejection-class response (e.g., SIP 603 Call Rejected).

If either phase fails to produce the expected response within the vouch-call timeout (or is missing, altered, or inconsistent), the entire vouch MUST be treated as unsuccessful or indeterminate.

### Vetting Mechanism

CIDVV uses a three-step handshake for vetting. All calls use the `101` prefix in the Calling Party Number.

- **Wake Call** (Alice -> Bob): Alice initiates using her vetting Caller-ID.
- **Recognize Call** (Alice -> Bob): Alice uses the Recognize Token as the Calling Party Number.
- **Auth Call** (Bob -> Alice): Bob uses the Auth Token as the Calling Party Number.

A successful **Vet** requires all three steps to complete successfully within the Validity Window.

The **Recognize Call** serves as critical anti-goading protection. Bob's platform will only initiate the final Auth Call if it has recently received a valid Wake Call *and* the Recognize Token matches what it computed. This prevents an attacker from tricking Bob's platform into calling Alice.

Both sides independently compute the short-lived Recognize Token and Auth Token from the shared secret and the two telephone numbers involved. The tokens are valid only within the Validity Window.

### Token Computation Algorithm (Normative)
{: #token-computation }

1. Normalize both telephone numbers to E.164 digit strings (no leading "+", no punctuation) as defined in Section [Number Normalization](#number-normalization).

2. For the **Recognize Token**:
   - Concatenate as UTF-8 bytes: `normalized-calling-number || "|" || normalized-called-number || "|" || shared-secret`

3. For the **Auth Token**:
   - Concatenate as UTF-8 bytes: `shared-secret || "|" || normalized-called-number || "|" || normalized-calling-number`

4. Compute the SHA-256 digest of the concatenated bytes.
5. Take the first 8 hexadecimal characters of the digest.
6. Convert that 8-hex string to a decimal integer.
7. Left-pad with zeros to 10 digits if needed, then prepend '1' to produce an 11-digit token.

**Security Note for Cloud Providers**:
Cloud-based CIDVV services MUST isolate customers (e.g., by including a unique Customer ID or Tenant ID in the token computation) to prevent one customer from successfully vetting another customer's numbers. Failure to properly partition customers would allow a malicious actor who knows one customer's shared secret to impersonate them.

**Example (for illustration only)**
- Calling number: `+12125550100`
- Called number: `+19495550199`
- Shared secret: `elephant`

Recognize Token calculation:
`12125550100|19495550199|elephant` -> (SHA-256 processing) -> `12953388433`

Auth Token uses the reversed number order after the shared secret.

Implementations MUST use identical normalization, concatenation order, and processing on both sides of the vetting exchange. Tokens are valid only inside the Validity Window.

### Detailed Vouching Procedure

When Alice wants to place a call to Bob using her asserted caller-id, the following steps are performed:

1. Alice's CIDVV platform is notified of her outbound call attempt to Bob (using her Asserted Caller-ID). This notification is typically done by Alice's SBC or gateway sending an INVITE to the CIDVV platform. The CIDVV platform rejects this INVITE with a SIP 404 (Not Found) response so that the original call can continue toward the PSTN. Note that the exact notification mechanism is implementation-specific and may use other methods (e.g., an API call or STIR signing request) in future deployments.

2. Bob's CIDVV platform intercepts the incoming call from Alice and holds it (does not yet alert Bob's phone).

3. Bob's CIDVV platform initiates two short verification calls back to Alice (in either order or in parallel):

   a. One verification call with Calling Party Number prefixed by `+100`.

   b. One verification call with Calling Party Number prefixed by `+101`.

   Both calls are directed to Alice's Asserted Caller-ID.

4. Alice's CIDVV platform intercepts each verification call and performs the following actions:

   a. For the call with `+100` prefix: Rejects the call with SIP **486 Busy Here**.

   b. For the call with `+101` prefix: Rejects the call with SIP **603 Call Rejected**.

5. Bob's CIDVV platform evaluates the responses to both verification calls.

6. If both verification calls receive the expected responses (486 for `+100` and 603 for `+101`) within the Validity Window, Bob's CIDVV platform:

   a. Considers the vouching successful.

   b. Allows the original call from Alice to proceed and ring through to Bob.

7. If either verification call fails to receive the correct response, times out, or does not complete within the Validity Window, Bob's CIDVV platform:

   a. Considers the vouching failed.

b. May take any of the following actions on the original call (implementation specific):

   - Reject the call (e.g., with SIP 603 Call Rejected).
   - Route the call to Bob's voicemail.
   - Allow the call to ring through to Bob with a visual warning on the display (e.g., "Unverified Caller-ID").

### Detailed Vetting Procedure

When Alice wants to confirm that Bob controls a particular telephone number, the following protocol is used:

1. Alice and Bob share a secret (e.g., "elephant").

3. Alice and Bob agree on the caller-id that Alice will use to vet Bob's number.

5. Alice's CIDVV platform initiates a "Wake Call" to Bob by dialing Bob's number prefixed with `+101` and using Alice's agreed vetting caller-id.

6. Bob's CIDVV platform performs the following actions:

   a. Intercepts the incoming call.

   b. Recognizes Alice's vetting caller-id and identifies this as a "Wake Call".

   c. Computes a short-lived "Recognize Token" derived from Alice's number + Bob's number + the shared secret (example: 13928543029).

   d. Stores this token temporarily in memory.

   e. Rejects the call with a SIP 603 (Call Rejected) response.

   f. Considers this a successful "Wake Call" from Alice to Bob (step 1 of 3).

7. Alice's CIDVV platform performs the following actions upon receiving the 603 response:

   a. Considers Bob's CIDVV platform "Awake".

   b. Independently calculates the same "Recognize Token" (example: 13928543029).

   c. Calculates an "Auth Token" using a slightly different derivation from the shared secret + Bob's number + Alice's number (example: 19020621754).

   d. Stores the Auth Token temporarily in memory.

   e. Initiates a "Recognize Call" (step 2 of 3) to Bob using the Recognize Token as the caller-id, prefixed with `+101` (example: `+10113928543029`).

8. Bob's CIDVV platform performs the following actions upon receiving the Recognize Call:

   a. Intercepts the call.

   b. Verifies that the received caller-id matches the previously stored Recognize Token.

   c. Considers this a successful "Recognize Call" from Alice to Bob (step 2 of 3).

   d. Trusts Alice's CIDVV platform based on the matching token.

   e. Rejects the call with a SIP 486 (Busy Here) response.

   f. Computes the corresponding "Auth Token" (example: 19020621754).

   g. Initiates an "Auth Call" (step 3 of 3) to Alice using the Auth Token as the caller-id, prefixed with `+101` (example: `+10119020621754`).

9. Alice's CIDVV platform performs the following actions upon receiving the Auth Call:

   a. Intercepts the call.

   b. Verifies that the received caller-id matches the previously stored Auth Token.

   c. Considers this a successful "Auth Call" from Bob to Alice (step 3 of 3).

   d. Trusts Bob's CIDVV platform based on the matching token.

   e. Rejects the call with a SIP 486 (Busy Here) response.

   f. Considers the vetting procedure complete and successful.

10. Bob's CIDVV platform receives the 486 (Busy Here) response from Alice and also considers the vetting procedure successful.

Only the legitimate owner of Bob's number can receive the token and cause the
correct response sequence. Alice must be reachable, but this is acceptable for
vetting use cases.

### Signaling Prefixes and Call Types

CIDVV uses the following special prefixes in the Calling Party Number:

| Prefix | Call Type          | Direction     | Purpose                                      | Expected Response (by callee) |
|--------|--------------------|---------------|----------------------------------------------|-------------------------------|
| +100   | Vouch Phase 1      | Bob -> Alice   | Vouching verification (Phase 1)              | 486 Busy Here                |
| +101   | Vouch Phase 2      | Bob -> Alice   | Vouching verification (Phase 2)              | 603 Call Rejected            |
| +101   | Wake Call          | Alice -> Bob   | Initiate vetting and trigger token generation| 603 Call Rejected            |
| +101   | Recognize Call     | Alice -> Bob   | Prove knowledge of shared secret             | 486 Busy Here                |
| +101   | Auth Call          | Bob -> Alice   | Prove control of destination number          | 486 Busy Here                |

**Note:** All vetting-related calls use the `+101` prefix. Context is determined by the caller-id used (vetting caller-id vs. token value) and the current state maintained by the CIDVV platform.

## Response Semantics

Because intermediate SIP and SS7/TDM networks may translate,
modify, or replace response codes, implementations MUST interpret
responses based on behavioral class (e.g., "Busy"-class vs.
"Rejection"-class) rather than exact numeric values.

Implementations SHOULD use SIP 486 (Busy Here) and 603 (Call Rejected)
as the canonical representations of these behaviors where possible.

CIDVV requires that these two rejection behaviors remain
distinguishable across the signaling path. Environments that cannot
preserve this distinction may not support enhanced verification.

### Phase 1 Verification ("100" Prefix)

A call using the "100" prefix is the **Phase 1** verification call. It succeeds only if it receives a "Busy"-class response (e.g., SIP 486 Busy Here).

### Phase 2 Verification ("101" Prefix)

A call using the "101" prefix is the **Phase 2** verification call. It succeeds only if it receives a "Rejection"-class response (e.g., SIP 603 Call Rejected or SIP 403 Forbidden).

### Combined Phase Behavior (Required for Vouch Success)

A successful vouch or successful vet **requires both Phase 1 and Phase 2** to complete with their expected behaviors within the Validity Window.

Implementations MUST NOT treat a single phase as sufficient. If either phase fails, is missing, altered, delayed, or inconsistent, the result MUST be treated as unsuccessful or indeterminate.

# Protocol Operation

## Vouching Procedure

Alice's CIDVV platform receives an attempted call from Alice to Bob.
It MUST construct a CIDVV token as defined in Section <xref target="simple-overview"/>
by prefixing "100" to the dialed number.

The CIDVV platform MUST cache the call attempt using the tuple:

   (Called Number, CIDVV Token)

for the Validity Window.

The CIDVV platform then rejects the call with SIP response 486
(Busy Here).

Alice's SBC receives the 486 and advances the original call through
the PSTN toward Bob using the original Caller-ID.

When Bob's system receives the call, a CIDVV-aware network element
(e.g., SBC) initiates a verification call toward Alice.

### Phase 1 Verification ("100")

Bob's CIDVV-aware element constructs the CIDVV token using the same
method (prefix "100" plus the rightmost 12 digits of the dialed
number) and initiates a verification call toward Alice using that
value as the Calling Party Number.

When Alice's SBC receives a call with a Calling Party Number
beginning with "100", it MUST route the call to the CIDVV platform.

Upon receiving the verification call, Alice's CIDVV platform MUST
look up the tuple:

   (Called Number, CIDVV Token)

cached for the Validity Window.

If a matching cache entry exists, the CIDVV platform MUST reject the
verification call with SIP response 486 (Busy Here).

If no matching cache entry exists, the CIDVV platform MUST reject the
verification call with SIP response 404 (Not Found).

A successful "100" verification (i.e., receipt of 486) indicates that
the originating party can receive calls at the Asserted Caller-ID
and constitutes a valid baseline vouch.

### Phase 2 Verification ("101")

Bob's CIDVV-aware element MAY initiate a second verification call
using a CIDVV token constructed by prefixing "101" to the same
12-digit payload.

When Alice's SBC receives a call with a Calling Party Number
beginning with "101", it MUST route the call to the CIDVV platform.

Upon receiving such a call, the CIDVV platform MUST reject the call
with SIP response 404 (Not Found), unless the call corresponds to an
active vetting procedure (see Section <xref target="vetting-procedure"/>).

A "101" verification call does not require cache lookup for vouching
purposes and MUST NOT be used as a standalone indicator of a
successful vouch.

### Combined Phase Behavior (Required for Success)

A successful CIDVV vouch **requires both** Phase 1 and Phase 2 verification calls to complete with their expected responses within the Validity Window.

- **Phase 1** ("100" prefix) MUST return a Busy-class response (e.g., SIP 486 Busy Here or 600 Busy Everywhere).
- **Phase 2** ("101" prefix) MUST return a Not-Found-class response (e.g., SIP 404 Not Found or 608 Rejected).

The two verification calls MAY be performed in any order or in parallel. Implementations MUST NOT assume a specific ordering.

If either Phase 1 or Phase 2 fails to produce the expected response (or is missing, delayed beyond the Validity Window, or altered), the entire vouch MUST be treated as unsuccessful or indeterminate.

The same combined Phase 1 + Phase 2 requirement applies to successful vetting, with Phase 2 semantics adjusted for token confirmation (see Vetting Procedure).

### Vouch Call Timers

The originating platform uses the **Validity Window** to determine how
long it will wait for the return vouch call(s).

Independently, both the originating and terminating platforms should
implement configurable local timers that control how long they wait
for signaling responses during each vouching call.

A default timeout of 3-6 seconds is reasonable for domestic calls.
For international destinations, longer timeouts (typically 8-20 seconds)
are recommended to accommodate higher Post-Dial Delay (PDD).

The two vouch calls (Phase 1 and Phase 2) may be initiated sequentially
or simultaneously.

## Correlation Model

CIDVV vouching correlates calls using the Asserted Caller-ID,
the called number, and a Validity Window. It does not attempt to
identify individual call legs across the PSTN.

If multiple calls with the same Asserted Caller-ID and called
number occur within the cache interval, implementations MAY treat
them as a single aggregate vouching state or MAY maintain a count of
pending attempts.

A successful vouch indicates that at least one matching call attempt
occurred during the Validity Window, rather than proving a
one-to-one correspondence between specific call legs.

## Hash Function for Vetting and State Storage
{: #hash-function }

The same deterministic algorithm MUST be used for:
1. Vetting token computation.
2. Any short-term cache (e.g., Redis) that stores vouching or vetting state.

**Algorithm (normative)**

1. Normalize both numbers: E.164 digit string, no leading "+", no punctuation (see Section <xref target="number-normalization"/>).
2. Concatenate as UTF-8 bytes: `normalized-calling-number || "|" || normalized-called-number || "|" || shared-secret`
3. Compute SHA-256 digest of the concatenated bytes.
4. Take the first 8 hexadecimal characters of the digest.
5. Convert that 8-hex string to a decimal integer.
6. Left-pad with zeros to 10 digits if needed, then prepend '1' to produce an 11-digit token.

Example (for illustration only):
- calling = 12125550100, called = 19495550199, secret = "elephant"
- Concatenated: "12125550100|19495550199|elephant"
- SHA-256 first 8 hex -> decimal -> padded/prepended token = 12953388433 (or similar)

Implementations MUST use the identical normalization and concatenation order for both vetting calls and any Redis (or equivalent) cache lookups. The token is valid only inside the Validity Window.

### Multi-Tenant Considerations

CIDVV platforms that perform vouching on behalf of multiple
independent customers MUST ensure that correlation state is scoped
per customer. This prevents unintended interaction between unrelated
vouching operations that may produce identical CIDVV payload values.

Implementations MAY use separate storage, partitioning, or
customer-specific identifiers to achieve this isolation.

This requirement does not apply to vetting operations, which are
already scoped by the shared secret.

## Vetting Procedure
{: #vetting-procedure }

Vetting a remote number requires three separate calls (distinct SIP
dialogs) using a pre-agreed shared key. The process confirms that
the **called party (Bob)** controls the target telephone number and
possesses the correct shared secret. In the examples below, Alice is
the verifier who initiates the three calls to Bob's number in order to
vet Bob's number.

Before vetting begins, Alice and Bob agree on a shared secret, Bob's
vetting Caller-ID, and a Validity Window.

Alice places a vetting call to Bob using a Caller-ID beginning with the digits "101".

When Bob's CIDVV platform receives the first vetting call, it removes
the "101" prefix and verifies that the resulting Caller-ID is expected
for the current vetting attempt.

Bob's platform MUST compute the vetting token using the algorithm
defined in Section <xref target="hash-function"/> and store the
resulting token for the Validity Window. It then rejects the call with
SIP response 603 (Decline).

Alice performs the same SHA-256 calculation and places a second vetting call to Bob. This second call uses a Caller-ID beginning with the Vetting Token Check prefix of "101" followed by the computed numeric code.

When Bob's CIDVV platform receives the Vetting Token Check call, it removes the "101" prefix and compares the remaining numeric code to the recently cached value.

If the numeric code matches, Bob's CIDVV platform MUST reject the call with SIP response 486 (Busy Here). Alice's platform treats this response as a successful Recognize.

When Bob's CIDVV platform receives the 486 response to the Regognize call, it places the Auth call to Alice using the Auth token as the Caller-ID.

If the numeric code matches, Alice's CIDVV platform MUST reject the call with SIP response 486 (Busy Here). Bob's platform treats this response as a successful Auth

Any other response, timeout, code mismatch, expired cache entry, or unexpected Caller-ID MUST be treated as an unsuccessful vet.

# Examples

## Successful Vouch Call Flow

The following diagram shows a successful vouch.

~~~~
  Alice    CIDVV_A    SBC_A      PSTN     SBC_B    CIDVV_B     Bob
    |----- INVITE ----->|         |         |         |         |
    |         |<-INVITE-|         |         |         |         |
    |         |- 404 -->|         |         |         |         |
    |         |         |-INVITE->|         |         |         |
    |         |         |         |-INVITE->|         |         |
    |         |         |         |         |-INVITE->|         |
    |         |         |         |         |<- +100 -|         |
    |         |         |         |<- +100 -|         |         |
    |         |         |<- +100 -|         |         |         |
    |         |<- +100 -|         |         |         |         |
    |         |- 486 -->|         |         |         |         |
    |         |         |- 486 -->|         |         |         |
    |         |         |         |- 486 -->|         |         |
    |         |         |         |         |- 486 -->|         |
    |         |         |         |         |<- +101 -|         |
    |         |         |         |<- +101 -|         |         |
    |         |         |<- +101 -|         |         |         |
    |         |<- +101 -|         |         |         |         |
    |         |- 603 -->|         |         |         |         |
    |         |         |- 603 -->|         |         |         |
    |         |         |         |- 603 -->|         |         |
    |         |         |         |         |- 603 -->|         |
    |         |         |         |         |<- 302 --|         |
    |         |         |         |         |----- INVITE ----->|
~~~~
{: #fig-successful-vouch title="Example Successful Vouch"}

In the diagram
 - "+100" represents a verification call whose Calling Party Number begins with the prefix "100" (or "+100") followed by Bob's Caller-ID
 - "+101" represents a verification call whose Calling Party Number begins with the prefix "101" (or "+101") followed by Bob's Caller-ID

### Successful Vouch Step-by-Step Description

The diagram above shows the high-level message flow. The following numbered steps provide the detailed behavior, including Caller-ID manipulation performed by CIDVV platforms.

Note that the two verification calls (Phase 1 and Phase 2) MAY be performed in either order or in parallel.

1. The originating user (Alice, Asserted Caller-ID `+12125550100`) initiates a call to Bob (`+19495550199`).

2. The call is routed from Alice's User Agent to her SBC, which forwards it to the originating CIDVV platform (CIDVV_A).

3. **CIDVV_A**:
   - Caches the call attempt using the tuple `(Calling: 12125550100, Called: 19495550199)` for the Validity Window.
   - Rejects the call with SIP **404 Not Found**.

4. Alice's SBC receives the 404 and advances the original call toward the PSTN using Alice's original Caller-ID.

5. The call reaches Bob's SBC via the PSTN. Because this is a new dialog (initial INVITE), Bob's **CIDVV-aware SBC** forwards the call to the terminating CIDVV platform (CIDVV_B).

6. **CIDVV_B** initiates Phase 1 and/or Phase 2 verification calls toward Alice's number (`+12125550100`):
   - Phase 1: Caller-ID = `+10019495550199`
   - Phase 2: Caller-ID = `+10119495550199`

7. Each verification call arrives at Alice's SBC via the PSTN.

8. **Alice's SBC** detects the leading `+100` or `+101` prefix and routes the call to CIDVV_A.

9. **CIDVV_A**:
   - Receives the call and looks up the cached tuple for the Validity Window.
   - For a `+100` prefix: Returns SIP **486 Busy Here** (successful Phase 1).
   - For a `+101` prefix: Returns SIP **603 Decline** (successful Phase 2).

10. **CIDVV_B** receives the expected responses for both phases. Once both Phase 1 and Phase 2 have succeeded, it considers the combined result a **Successful Vouch**.

11. CIDVV_B returns a SIP **302 Moved Temporarily** (pointing to Bob's Contact) to Bob's SBC.

12. Bob's SBC receives the 302 and advances the original call to Bob's User Agent.

13. Bob's telephone rings.

This mechanism allows the originating CIDVV platform to confirm control of the Asserted Caller-ID without completing the initial call to Bob.

## Unsuccessful Vouch

A vouch attempt is considered unsuccessful or indeterminate if the
expected verification behavior is not observed.

Specifically:

* If a verification call using the "100" prefix does not result in
  SIP 486 (Busy Here), the vouch MUST be treated as unsuccessful.

* If both verification calls are performed, and the expected pattern
  of:
    - "100" -> 486, and
    - "101" -> 603
  is not observed within the Validity Window, the vouch MUST be
  treated as unsuccessful or indeterminate.

Implementations MUST fail closed. Any ambiguity, unexpected response,
timeout, or call progression MUST result in an unsuccessful or
indeterminate outcome.

## Vetting a Caller-ID Number

Vetting uses three independent verification calls that form a
challenge-response sequence. For clarity, the calls are shown
separately, but together they constitute a single vetting operation.

### First Vetting Call (Wake) using known Vetting number as Caller-ID

~~~~
   CIDVV_A        SBC_A          PSTN         SBC_B        CIDVV_B
      |             |             |             |             |
      |-- WAKE101 ->|             |             |             |
      |             |-- WAKE101 ->|             |             |
      |             |             |-- WAKE101 ->|             |
      |             |             |             |-- WAKE101 ->|
      |             |             |             |             |
      |             |             |             |<--- 603 ----|
      |             |             |<--- 603 ----|             |
      |             |<--- 603 ----|             |             |
      |<--- 603 ----|             |             |             |
      |             |             |             |             |
~~~~
{: title="Wake vetting call with 101 - Bob creates Recognize Token and responds with 603"}

### Second Vetting Call using Recognize Token as Caller-ID

~~~~
   CIDVV_A        SBC_A          PSTN         SBC_B        CIDVV_B
      |             |             |             |             |
      |-- RECG101 ->|             |             |             |
      |             |-- RECG101 ->|             |             |
      |             |             |-- RECG101 ->|             |
      |             |             |             |-- RECG101 ->|
      |             |             |             |             |
      |             |             |             |<--- 486 ----|
      |             |             |<--- 486 ----|             |
      |             |<--- 486 ----|             |             |
      |<--- 486 ----|             |             |             |
      |             |             |             |             |
~~~~
{: title="Recognize Call (101 prefix) - Bob acknowledges valid Recognize Token with 486 Busy Here"}

### Third Vetting Call (Auth) using Auth Token as Caller-ID

~~~~
   CIDVV_A        SBC_A          PSTN         SBC_B        CIDVV_B
      |             |             |             |             |
      |             |             |             |<- AUTH101 --|
      |             |             |<- AUTH101 --|             |
      |             |<- AUTH101 --|             |             |
      |<- AUTH101 --|             |             |             |
      |             |             |             |             |
      |---- 486 --->|             |             |             |
      |             |---- 486 --->|             |             |
      |             |             |---- 486 --->|             |
      |             |             |             |---- 486 --->|
      |             |             |             |             |
~~~~
{: title="Auth Call (101 prefix) - Alice acknowledges valid Auth Token with 486 Busy Here"}

### Successful Caller-ID Vetting Flow

Vetting a remote number requires three separate calls (distinct SIP dialogs) that together form a single challenge-response operation protected by a pre-agreed shared secret. The process confirms that the called party (Bob) controls the target telephone number and possesses the correct shared secret.

1. Alice and Bob agree on a shared secret (e.g., `elephant`) and Alice's vetting Caller-ID (e.g., `+12125550100`).

2. Both parties configure their CIDVV platforms with the shared secret, Alice's vetting Caller-ID, and an **optional validity period** (e.g., “one week”). This allows the number owner (Bob) to limit how long the shared secret remains active for vetting.

3. Alice's CIDVV platform (CIDVV_A) initiates the **Wake Call** to Bob's number (`+19495550199`) with Caller-ID `+10112125550100`.

4. Bob's SBC recognizes the `101` prefix and forwards the call to CIDVV_B.

5. **CIDVV_B**:
   - Strips the `101` prefix to recover Alice's vetting Caller-ID.
   - Recognizes it as a pre-agreed vetting number.
   - Computes the Recognize Token.
   - Caches the token for the Validity Window.
   - Rejects the call with SIP **603 Decline**.

6. CIDVV_A receives the 603 and computes the same Recognize Token.

7. CIDVV_A immediately places the **Recognize Call** to Bob using Caller-ID `+101` followed by the Recognize Token (e.g., `+10112953388433`).

8. Bob's SBC forwards the call (based on the `101` prefix) to CIDVV_B.

9. **CIDVV_B**:
   - Strips the `101` prefix.
   - Verifies the token matches the cached value.
   - Responds with SIP **486 Busy Here**.

10. CIDVV_A receives the 486 and computes the Auth Token.

11. CIDVV_B computes the Auth Token and initiates the **Auth Call** to Alice's number using Caller-ID `+101` followed by the Auth Token.

12. **CIDVV_A**:
    - Strips the `101` prefix.
    - Verifies the Auth Token.
    - Responds with SIP **486 Busy Here**.
    - Declares the vetting successful.

13. CIDVV_B receives the 486 and also declares the vetting successful.

All calls are short signaling-only exchanges. The entire operation MUST complete within the Validity Window.

#### Vetting Failure Cases

A vetting attempt may fail for the following reasons:

* Bob does not have a participating CIDVV platform - the first call will not return 603, or the second call will not return 486.
* The shared secret, Alice's vetting Caller-ID, or time window does not match - the two calls will not produce the expected 603 + 486 sequence.
* Network or policy restrictions prevent one or both calls from reaching the remote CIDVV platform.

In all such cases, the vetting attempt MUST be treated as unsuccessful.

This three-call challenge-response mechanism provides strong confirmation that the remote number is both reachable via the PSTN and controlled by an entity that knows the shared secret.

# Deployment Considerations

## Behavior of Non-CIDVV Systems

Systems that do not implement CIDVV are not expected to recognize the
CIDVV signaling prefixes. Such systems will typically process these
calls as ordinary calls and may return a wide range of responses.
CIDVV implementations MUST treat any response that does not match the
expected protocol behavior as indicating a non-participating system.

## Handling of CIDVV Signaling Calls

Networks and SBCs that recognize CIDVV signaling SHOULD intercept calls
with Calling Party Numbers beginning with "100", "101", "+100", or "+101"
before they reach end users. These calls SHOULD result in a non-success
response (commonly 486 Busy Here or 603 Decline) and MUST NOT establish
media.

Call analytics, labeling, and fraud detection systems SHOULD recognize
these prefixes and treat the calls as protocol signaling rather than
ordinary subscriber traffic.

## Carrier Incentives and SBC Policies

The short-lived CIDVV signaling calls represent a very small incremental
load on the network. In return, carriers gain a powerful mechanism to
reduce the much larger burden of fraudulent and nuisance calls.

Carriers and SBC operators are strongly encouraged to implement the
following simple rule for calls where the Calling Party Number begins
with `+100`, `+101`, `100`, or `101`:

- If a `200 OK` response is received, convert it to `603 Decline`.

This single rule prevents media establishment and billing while allowing
the signaling call to reach the destination CIDVV platform. It can be
implemented with a simple, stateless prefix match on the Calling Party
Number at originating, border, and transit SBCs.

## Response Variability

Implementations SHOULD interpret responses based on behavioral class
(e.g., "busy" class such as 486 Busy Here versus "rejection" class such
as 603 Decline or 403 Forbidden) rather than relying on exact numeric
values. Intermediate networks may translate or modify response codes,
so behavioral class is the preferred signal.

## Short-Term State Management

CIDVV relies on short-lived state for the (Calling Number, Called Number)
tuple, valid only for the Validity Window. Implementations MUST expire
this state automatically and MUST fail closed: on restart or state loss,
treat all verification requests as unsuccessful until fresh state has
been deposited.

## International and Cross-Border Operation

International calls often experience higher Post-Dial Delay. Vouching
call timeouts should be adaptive: 3-6 seconds is appropriate for domestic
calls, while 8-20 seconds (or more) may be needed for international
destinations based on observed PDD. The two vouch calls (Phase 1 and
Phase 2) may be performed sequentially or simultaneously.

# Operational Considerations

## Protocol Operation - Vouching

If a vouching call results in a provisional response (e.g., 180
Ringing) or a successful response (200 OK), the originating system
SHOULD immediately cancel the call and treat the remote system as not
implementing CIDVV.

## Failure and Restart Behavior

CIDVV platforms rely on short-lived state. Upon restart or loss of
state, implementations SHOULD continue accepting new call deposits
but MUST treat all verification requests as unsuccessful until sufficient
state has been rebuilt.

Implementations SHOULD return a non-success response (e.g., 4xx,
5xx, or 6xx). A 603 (Decline) response is commonly used to indicate
that verification could not be performed.

Implementations SHOULD fail closed (treating requests as unverified)
rather than risk false-positive validation.

## Number Normalization
{: #number-normalization }

All telephone numbers used in CIDVV operations (for caching, token generation, comparison, etc.) MUST be normalized to a plain digit string in E.164 format **without any leading "+" sign** as follows:

1. Remove any leading "+" or other punctuation characters (parentheses, dashes, spaces, etc.).
2. Use the full E.164 representation: country code followed by the national significant number.
3. No padding is performed. If truncation is required to stay within the 15-digit limit for the Calling Party Number field, always remove leading digits (preserving the rightmost digits).

**Note**: While the leading `+` may be present in SIP signaling or user-facing displays, it MUST be stripped before any CIDVV processing, tuple storage, or token computation.

## Prefix Preservation

Because CIDVV signaling is carried entirely in the **Calling Party Number** (using the `+100` and `+101` prefixes), it is critical that these prefixes survive network traversal.

SIP intermediaries, SBCs, and PSTN gateways **SHOULD** preserve the full Calling Party Number, including the leading `100` or `101` prefix, when forwarding CIDVV signaling calls across trusted interfaces.

If a network element cannot preserve the prefix, it SHOULD reject the call with a 603 (Decline) response rather than forwarding a modified version that would break the protocol.

## Interaction with Call Analytics and Fraud Detection

CIDVV signaling calls use Calling Party Number values that may appear
anomalous to call analytics, labeling, and fraud detection systems.

Systems that support such analytics SHOULD recognize CIDVV signaling
prefixes (e.g., "100" and "101") and treat such calls as protocol
signaling rather than ordinary subscriber traffic.

CIDVV signaling calls are not intended to be presented to end users
and SHOULD NOT be labeled or blocked as malicious traffic when
processed within cooperating networks.

Failure to recognize CIDVV signaling may result in increased false
positives or suppression of verification attempts.

## Edge Cases and Special Handling

### Multiple Simultaneous Calls from the Same Caller-ID (Fan-Out)

In some scenarios, multiple calls may legitimately originate from the same Asserted Caller-ID in a short period of time (e.g., an office full of people calling a radio contest to be the "100th caller", a call center, or a political phone bank).

In such cases, the originating CIDVV platform **MAY** switch to "multi-call" mode for that (Asserted Caller-ID, Called Number) tuple:

- Instead of enforcing strict 1:1 correlation, the platform treats the tuple as active for the Validity Window.
- Each new outbound call from that Caller-ID restarts (or extends) the expiration timer.
- Verification calls (Phase 1 and Phase 2) do **not** consume or delete the state, allowing multiple parallel or closely spaced calls to be vouched successfully.

This "multi" token mode is an implementation-specific optimization. Platforms that do not support it MAY simply reject or rate-limit excessive concurrent calls from the same number.

### Switched Toll-Free Caller-ID

Some toll-free numbers are configured as "switched toll-free." In these cases, the PSTN translates the toll-free number into a regular DID before delivering the call to the subscriber's PBX or SIP trunk. The called party (or their CIDVV platform) sees the call arriving at the mapped DID rather than the original toll-free number.

When the Asserted Caller-ID is a switched toll-free number, the originating CIDVV platform **MUST** be aware of the toll-free -> DID mapping. It SHOULD create cache entries for **both** numbers:

- The toll-free number (as presented to the called party in the original call)
- The mapped DID (where the verification calls will actually be delivered)

This ensures the platform can correctly respond to verification calls arriving at either number.

Platforms that do not support switched toll-free mappings will be unable to successfully vouch calls using those toll-free numbers as the Asserted Caller-ID.

### Call Forwarding / Diversion

When the called party (Bob) has forwarded their number to a different destination (Charlie), the verification calls must still be directed to the original Asserted Caller-ID (Bob's number), **not** the final diverted destination.

- The terminating CIDVV platform (or CIDVV-aware SBC) at Charlie's location SHOULD use the original called number (Bob's number) from the diversion / redirection information (e.g., SIP `Diversion` header or ISDN Redirecting Number field).
- Alice's CIDVV platform will receive verification calls to Bob's number (which it is expecting) and can respond appropriately.
- Charlie's platform is responsible for performing the vouch on behalf of the forwarded number.

This ensures the vouch reflects control of the number the caller originally asserted (Bob's number), even if the call was ultimately delivered elsewhere.

### Numbers That Should Never Be Vouched (Low-Latency Bypass)

Certain numbers or destinations require extremely low call-setup latency and cannot tolerate the additional delay of CIDVV verification (typically 3-8 seconds round-trip).

Examples include:
- Trading floors and high-frequency trading desks
- Emergency operations centers
- Alarm monitoring systems
- Certain government and military command lines

CIDVV platforms **SHOULD** support a configurable "Never Vouch" list (or whitelist for immediate pass-through). When an incoming call matches an entry in this list, the platform MUST forward the call immediately without attempting any verification calls.

This decision can be implemented at the CIDVV platform itself or at a CIDVV-aware SBC.

# Security Considerations

CIDVV verification is probabilistic and based on reachability rather than cryptographic identity. It is intended to complement, not replace, mechanisms such as STIR/SHAKEN.

**Important Limitation**
CIDVV specifically addresses Caller-ID spoofing and impersonation. It does **not** prevent all forms of telephone fraud. Scammers who call from numbers they legitimately control (or that a malicious or compromised service provider controls) can still obtain successful vouches. A malicious CIDVV platform could also "fail open" and vouch for every call. These attacks are outside the scope of the spoofing protection CIDVV provides.

Its security properties derive primarily from the difficulty an attacker faces in receiving calls at the Asserted Caller-ID (the number being vouched).

CIDVV validates reachability within a short Validity Window rather than providing strict per-call correlation. This may result in multiple calls being validated by a single successful vouch.

The use of distinct response patterns across the two verification calls (Phase 1 with "100" prefix expecting 486 Busy Here, Phase 2 with "101" prefix expecting 603 Decline) increases resistance to false-positive validation caused by common network behaviors.

## Trust Model

CIDVV assumes that:
- The PSTN routes calls to the correct terminating service provider
  for a given telephone number.
- The terminating service provider has authoritative control over the
  number and can originate return calls.
- Intermediate networks may modify signaling but will generally
  preserve sufficient information to allow correlation of requests
  and responses.

CIDVV does not assume that Caller-ID values are trustworthy; instead,
it verifies control through network reachability.

## Replay and Ride-Along Attacks

CIDVV relies on short-lived state to correlate signaling exchanges. This significantly limits the window for replay and ride-along attacks.

**Replay Attacks**
An attacker who observes a successful verification exchange cannot effectively replay it after the cached state expires. Implementations **MUST** expire cached state quickly. A recommended default is **10 seconds**, although longer values (e.g., 15-30 seconds) MAY be used for international or high-latency routes.

Implementations **MUST** reject verification attempts that do not match recent, valid cached state.

**Ride-Along Attacks**
A scammer who successfully obtains a vouch for one call might attempt to "ride along" and use the same vouch for additional calls. Because the originating CIDVV platform controls when cached state is deleted, implementers have flexibility in mitigation:

- An implementation **MAY** delete the cached state immediately after successfully processing the Phase 1 ("100") verification call and returning the 486 response.
- This approach greatly reduces the opportunity for ride-along attacks while still allowing legitimate parallel verification calls.

The choice of when to expire or delete state is left to the implementer, as it involves a trade-off between security and operational robustness (e.g., handling delayed or out-of-order verification calls).

## Spoofing Resistance

CIDVV prevents spoofing by requiring the party asserting a Caller-ID
to successfully receive and respond to a return call routed via the
PSTN. An attacker (Mallory) who does not control the corresponding
number cannot receive the verification call and therefore cannot
complete the vouching process.

## Denial of Service

CIDVV introduces additional signaling traffic, which could be abused for denial-of-service attacks.

Implementations MUST:
- Rate-limit CIDVV signaling requests per source and per destination.
- Detect and suppress repeated unsuccessful attempts from the same source.
- **Bound resource usage for temporary state** (e.g., limit the maximum number of concurrent cache entries and automatically expire old entries).

Implementations SHOULD:
- Apply per-source and per-destination limits
- Monitor for anomalous traffic patterns

## Amplification and Reflection

CIDVV generates return calls as part of its operation. Care MUST be
taken to ensure that this behavior cannot be exploited for
amplification or reflection attacks.

Implementations SHOULD:
- Only initiate return calls in response to valid inbound attempts
- Limit the rate of outbound verification calls
- Avoid generating multiple responses for a single triggering event

## Response Code Manipulation

CIDVV does not require specific SIP response codes to be preserved
end-to-end, but it does require that distinct rejection behaviors
(e.g., "busy" vs. "reject") remain distinguishable.

Implementations MUST interpret responses based on behavioral class
(e.g., "Busy"-class vs. "Rejection"-class) rather than exact numeric values.

## Data Privacy

CIDVV exchanges inherently expose calling and called numbers within
signaling messages.

Implementations SHOULD:
- Avoid storing telephone numbers in plaintext where possible
- Use derived values (e.g., cryptographic hashes) for temporary state
- Limit retention of any identifying data

Temporary state MUST be short lived and automatically expired.

## Hash-Based Token Security (Vetting)

The security of the vetting mechanism depends on the shared secret and the derived Recognize and Auth Tokens.

Implementations MUST:
- Use a cryptographically secure hash function (SHA-256 is REQUIRED).
- Protect shared secrets from disclosure.
- Ensure that tokens are only valid within the Validity Window and are automatically expired.

Implementations SHOULD:
- Use sufficiently long and random shared secrets.
- Avoid reusing the same secret across many numbers or for long periods.

## Failure Modes

CIDVV implementations MUST fail closed. If verification cannot be
completed due to:
- network errors
- state loss
- unexpected responses

the result MUST be treated as unverified.

## Interoperability Risks

CIDVV operates across heterogeneous networks, including SIP and
SS7/TDM environments. Intermediate systems may:
- modify Calling Party Number values
- truncate digits
- alter signaling behavior

These behaviors may cause verification to fail but MUST NOT result in
false-positive validation.

## Residual Risk

CIDVV improves resistance to Caller-ID spoofing but does not provide
absolute identity assurance. It reduces the effectiveness of spoofing
attacks rather than eliminating them and relies on probabilistic
verification based on reachability and response behavior, not
cryptographic identity binding.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

The authors thank contributors to telephony security research and PSTN infrastructure development.

# Appendix: Changes from Previous Version

RFC Editor: Please remove this section before publication as an RFC.

## Changes since draft-anderson-askew-cidvv-00 (May 2026)

- Minor editorial improvements and clarifications throughout the document.
- Made both Vouching calls required and renamed them as "Phase 1" and "Phase 2" for clarity.
- Updated termination handling so that specific response codes (486 Busy Here, 603 Decline, and 403 Forbidden) are used to prevent SBCs from advancing to the next route.
- Added a third call to the Vetting process to ensure Alice and Bob each trust the other shares the key.
- Added this "Changes from Previous Version" appendix to support long-term document maintenance across multiple revisions.
