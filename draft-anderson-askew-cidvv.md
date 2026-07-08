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

CIDVV is complementary to STIR/SHAKEN and other identity frameworks. It provides an incrementally deployable tool that works even in environments where cryptographic attestation is not yet available or sufficient, while being designed to tolerate common forms of intermediate network modification.

By requiring demonstrable real-time control of the Asserted Caller-ID, CIDVV strengthens resistance to spoofing in a practical, low-overhead manner.

--- middle

# Introduction

CIDVV supports two closely related functions: **Vouching** (real-time verification that the asserted number vouches for a specific call) and **Vetting** (confirmation that a number is controlled by its expected owner, useful for branding, trust programs, and registries). The primary focus of this document is the vouching mechanism, which directly addresses Caller-ID spoofing for individual calls.

Caller-ID spoofing remains a widespread problem in modern telephony. Fraudulent and nuisance callers frequently impersonate legitimate numbers, eroding trust and complicating call screening for recipients.

At the signaling layer, legitimate Caller-ID use and malicious impersonation
can appear identical. Originating networks routinely assert a
Caller-ID on behalf of a calling party, but intermediate carriers and
terminating networks generally cannot infer the **intent** behind that
assertion from ordinary PSTN signaling alone. Without a reliable identity
signal, the network may be unable to distinguish legitimate use
from malicious impersonation.

This document defines **Caller-ID Vouching and Vetting (CIDVV)**, a lightweight, incrementally deployable mechanism that allows the called party, or a platform acting on its behalf, to ask a simple real-time question:

> "Will the party responsible for this number vouch for this call right now?"

CIDVV verifies Caller-ID control through network reachability rather than relying solely on asserted identity. It requires that a party asserting a Caller-ID demonstrate control of that number by being able to receive a short return signaling call within a brief Validity Window.

CIDVV operates by encoding signaling information within the Calling Party Number and leveraging existing call routing behavior to perform a challenge-response exchange. The protocol requires no new SIP headers, protocol extensions, response codes, or changes to SS7 signaling. It is designed to function across mixed SIP and TDM networks, including international paths.

**CIDVV is complementary to STIR/SHAKEN** and other identity frameworks. It provides practical protection in environments where cryptographic attestation is not yet fully deployed, while being designed to tolerate common signaling modifications by intermediate networks. It intentionally uses distinct failure-response behaviors as part of its signaling mechanism and does not require universal adoption to deliver benefit.

The mechanism leverages two key elements of the existing telephone ecosystem:

* **Authoritative PSTN routing**: Calls to a telephone number are generally routed to the provider, service, or party responsible for that number. CIDVV uses this existing routing behavior to test whether the party responsible for an asserted Caller-ID can receive and respond to a return verification call.

* **Calling Party Number encoding**: CIDVV carries its signaling state in compact numeric values placed in the Calling Party Number. The prefixes "100" and "101" identify CIDVV verification calls while preserving ordinary routing to the asserted telephone number.

CIDVV operates entirely within standard PSTN routing behavior and requires no media exchange. While it does not provide absolute identity assurance, it delivers strong, real-time evidence of Caller-ID control in a practical and low-overhead manner.

# Terminology

* **Caller-ID**: The telephone number presented to the called party (what the end user sees).
* **Asserted Caller-ID**: The Caller-ID value that is being vouched or vetted by this protocol. This is the number whose control the calling party claims, and it is used for state management, token computation, and correlation.
* **Calling Party Number**: The value carried in the signaling protocol (e.g., SIP `From` header or ISUP Calling Party Number parameter). In many deployments this is the same as the presented Caller-ID, but they are not always identical.
* **Alice**: The calling party and verifier. In vouching flows Alice asserts a number; in vetting flows Alice verifies Bob's number.
* **Bob**: The called party. In vetting flows Bob is the owner whose number is being vetted.
* **CIDVV Platform**: A system that implements the vouching and vetting procedures defined in this document.
* **CIDVV-aware Network Element**: A network element (typically an SBC or proxy) that recognizes CIDVV signaling prefixes ("100" and "101") in the Calling Party Number and routes those calls to a CIDVV platform. In some deployments, it may also forward initial INVITEs for new dialogs to a CIDVV platform and handle local responses that allow the original call to continue.
* **Vouch**: The act of a CIDVV platform asserting that it has verified control of a telephone number through the challenge-response mechanism described in this document, which may consist of one or more verification calls. A successful vouch provides strong evidence that the calling party controls the Asserted Caller-ID.
* **Vet** (or **Vetting**): The process by which a CIDVV platform confirms that a party controls a telephone number via the three-call challenge-response sequence. Vetting may be performed on behalf of third parties such as Caller-ID branding services, Vetting Agents, law enforcement agencies, trade organizations, or enterprise trust programs.
* **Vouching Call**: A short signaling call used in the CIDVV protocol. CIDVV defines **Phase 1** ("100" prefix) and **Phase 2** ("101" prefix) verification calls.
* **Phase 1 Vouch** ("100" prefix): The initial Vouch verification step. Expected response behavior is a "Busy"-class response (e.g., SIP 486 Busy Here).
* **Phase 2 Vouch** ("101" prefix): The secondary Vouch step. Expected response behavior is a "Rejection"-class response (e.g., SIP 603 Decline).
* **Successful Vouch**: Requires **both Phase 1 and Phase 2** to complete with the expected behaviors within the Validity Window.
* **Verification Not Performed**: A condition where verification could not be completed due to system or network conditions.
* **Validity Window**: The time interval during which the originating CIDVV platform will accept and correlate a vouch attempt (return call) from the called party. This is typically on the order of 10-30 seconds.
* **Unsuccessful Vouch**: A verification result indicating that the vouch did not complete successfully, including cases involving missing state, unexpected response behavior, timeout, altered signaling, or incomplete verification phases.
* **Vouch-Call Timeout**: A local timer used by the platform that initiates a Phase 1 or Phase 2 verification call to limit how long it waits for a response. This is typically 3-6 seconds for domestic calls and longer (e.g., 8-20 seconds) for international calls. It is distinct from the Validity Window.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14, RFC 2119 and RFC 8174 when, and only when, they appear in all
capitals, as shown here.

## Motivation and Advantages

CIDVV was designed in response to practical constraints observed in
real-world telephony deployments. Earlier design alternatives included
carrying verification data in Real-Time Text (RTT), media, SIP headers,
or SDP bodies. These approaches were not selected because they either
required media-path establishment, depended on SIP-specific behavior, or
were likely to be removed, rewritten, or ignored by back-to-back user
agents, gateways, or interworking functions in mixed SIP, SS7/TDM, and
ISDN environments.

CIDVV instead carries short-lived verification state in the Calling Party
Number and uses distinguishable non-success response behaviors as the
verification signal. This design favors information elements and response
semantics that are commonly preserved across heterogeneous telephone
networks, including inter-provider and international paths. The result is
a mechanism that can be deployed incrementally without defining new SIP
headers, new SIP response codes, new media behavior, or new SS7/ISDN
protocol elements.

CIDVV is not intended to replace STIR/SHAKEN. STIR/SHAKEN provides
important cryptographic caller identity assurance and remains a key part
of the telephone identity ecosystem. CIDVV adds a complementary
reachability-based signal: whether the party responsible for the
Asserted Caller-ID can receive and correctly respond to verification
calls for this call attempt within the Validity Window.

The primary advantages of CIDVV are:

* **Uses existing telephony behavior**: CIDVV relies on existing call
  routing, Calling Party Number delivery, and non-success response
  behavior rather than new protocol extensions.

* **Provides reachability-based anti-spoofing evidence**: A successful
  vouch gives strong evidence that a party controlling the Asserted
  Caller-ID is participating in the call attempt during the Validity
  Window.

* **Works across mixed network environments**: CIDVV is designed for SIP,
  SS7/TDM, ISDN, and interworking environments, including paths where
  SIP-specific identity information may not survive end-to-end.

* **Requires no media exchange**: Verification calls are short
  signaling-only exchanges and are not intended to establish media.

* **Supports incremental deployment**: CIDVV can be implemented by
  enterprises, service providers, or third-party CIDVV platforms. Universal
  deployment is not required for cooperating parties to obtain benefit.

* **Provides operational visibility**: Number owners and their providers
  can gain telemetry about attempted use of their numbers in suspicious
  or spoofed calling scenarios.

* **Supports flexible operational models**: CIDVV can be deployed by the
  originating provider, terminating provider, enterprise SBCs, hosted
  platforms, or third-party services, depending on local policy and
  routing arrangements.

By avoiding dependence on centralized authorities or end-to-end SIP
feature preservation, CIDVV lowers the barrier to deployment while
providing a practical additional tool for detecting and reducing
Caller-ID spoofing.

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

* **Vouching** - Allows the called party (or their provider) to verify in real time whether the party responsible for the presented Caller-ID vouches for *this specific call*.

* **Vetting** - Allows confirmation that a telephone number is under the control of its expected owner. This is useful for Caller-ID branding services, enterprise trust programs, industry registries, and similar applications.

### Vouching Operation (Primary Use Case)

When Alice wants to place a call to Bob while asserting a particular Caller-ID:

1. Alice places a normal call to Bob using the Asserted Caller-ID. Alice's CIDVV platform is notified of the outbound call attempt.
2. Bob's CIDVV platform intercepts the incoming call before ringing Bob's phone.
3. Bob's platform initiates **two short signaling-only verification calls** back to Alice's Asserted Caller-ID (these may be performed in parallel):
   - **Phase 1** verification call using Calling Party Number prefix `"100"`.
   - **Phase 2** verification call using Calling Party Number prefix `"101"`.
4. Alice's CIDVV platform recognizes the special prefixes on the incoming verification calls and responds with the expected rejection behavior for each phase. This proves that it controls the Asserted Caller-ID **and** that Alice has an active call in progress to Bob.
5. Bob's CIDVV platform evaluates the verification result. If both Phase 1 and Phase 2 succeed within the Validity Window, the vouch is successful. If either phase fails, times out, or produces an unexpected response, the vouch is unsuccessful or indeterminate. The handling of the original call is implementation-specific; the platform may allow the call, label it, route it differently, send it to voicemail, or reject it according to local policy.

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

* **"100"** - Phase 1 Verification Call
* **"101"** - Phase 2 Verification Call

A successful **Vouch** requires **both** Phase 1 and Phase 2 to complete with their expected responses within the Validity Window. The two phases MAY be performed in any order or in parallel.

**Expected behaviors**:
* **Phase 1** ("100" prefix): MUST receive a Busy-class response (e.g., SIP 486 Busy Here).
* **Phase 2** ("101" prefix): MUST receive a Rejection-class response (e.g., SIP 603 Decline).

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

CIDVV vetting uses two derived numeric tokens:

* **Recognize Token**: Computed by Alice and verified by Bob.
* **Auth Token**: Computed by Bob and verified by Alice.

Both tokens are computed using the same processing steps, but with
different input ordering. This prevents the Recognize Token and Auth
Token from being interchangeable.

1. Normalize both telephone numbers to E.164 digit strings with no
   leading "+" and no punctuation, as defined in Section
   [Number Normalization](#number-normalization).

2. For the **Recognize Token**, concatenate the following values as
   UTF-8 bytes:

   `normalized-calling-number || "|" || normalized-called-number || "|" || shared-secret`

   In this context, `normalized-calling-number` is Alice's vetting
   Caller-ID and `normalized-called-number` is Bob's number being vetted.

3. For the **Auth Token**, concatenate the following values as UTF-8
   bytes:

   `shared-secret || "|" || normalized-called-number || "|" || normalized-calling-number`

4. Compute the SHA-256 digest of the concatenated bytes.

5. Take the first 8 hexadecimal characters of the digest.

6. Convert that 8-character hexadecimal string to a decimal integer.

7. Left-pad the decimal value with zeros to 10 digits if needed, then
   prepend the digit "1" to produce an 11-digit token.

**Example (for illustration only)**

- Alice vetting Caller-ID: `+12125550100`
- Bob's number: `+19495550199`
- Shared secret: `elephant`

Recognize Token input:

`12125550100|19495550199|elephant`

Auth Token input:

`elephant|19495550199|12125550100`

Recognize Token calculation:

- SHA-256 digest begins with: `b86a096e`
- First 8 hexadecimal characters: `b86a096e`
- Decimal value: `3093956974`
- 10-digit decimal value: `3093956974`
- Final 11-digit Recognize Token: `13093956974`

Auth Token calculation:

- SHA-256 digest begins with: `9f6b0648`
- First 8 hexadecimal characters: `9f6b0648`
- Decimal value: `2674591304`
- 10-digit decimal value: `2674591304`
- Final 11-digit Auth Token: `12674591304`

The resulting Recognize Token and Auth Token differ because the input
ordering is different.

Implementations MUST use identical normalization, input ordering, digest
processing, decimal conversion, zero-padding, and prefixing on both sides
of the vetting exchange. Tokens are valid only within the Validity Window
or, for multivetting, within the refreshed Validity Window.

Hosted or multi-tenant CIDVV services MUST ensure that shared secrets,
configuration, token state, and cached vetting state are scoped to the
appropriate customer or tenant. A tenant-specific value MAY be included
as an additional token-computation input only when both sides of the
vetting relationship are explicitly configured to use the same value.
Otherwise, tenant isolation MUST be enforced by provisioning, storage,
and access-control boundaries rather than by changing the token
algorithm.

### Detailed Vouching Procedure

When Alice wants to place a call to Bob using her Asserted Caller-ID, the following steps are performed:

1. Alice's CIDVV platform is notified of her outbound call attempt to Bob
   using her Asserted Caller-ID. The notification mechanism is
   implementation-specific. For example, Alice's SBC or gateway might
   send an INVITE to the CIDVV platform before advancing the original
   call toward the PSTN, or the platform might be notified through an API,
   routing policy, STIR signing workflow, or other local mechanism.

   Upon receiving this notification, Alice's CIDVV platform records the
   attempted call state for the Validity Window so that it can respond to
   subsequent Phase 1 and Phase 2 verification calls.

2. Bob's CIDVV platform intercepts the incoming call from Alice and holds it (does not yet alert Bob's phone).

3. Bob's CIDVV platform initiates two short verification calls back to Alice (in either order or in parallel):

   a. One verification call with Calling Party Number prefixed by `+100`.

   b. One verification call with Calling Party Number prefixed by `+101`.

   Both calls are directed to Alice's Asserted Caller-ID.

4. A CIDVV-aware network element serving Alice's Asserted Caller-ID
   recognizes the `+100` or `+101` prefix in the Calling Party Number and
   routes the verification call to Alice's CIDVV platform.

5. Alice's CIDVV platform receives each verification call and performs the following actions:

   a. For the call with `+100` prefix: Looks up the cached call state and rejects the call with SIP **486 Busy Here** if matching state exists.

   b. For the call with `+101` prefix: Rejects the call with SIP **603 Decline** for vouching purposes, unless the call corresponds to an active vetting procedure as described in Section [Vetting Procedure](#vetting-procedure).

6. Bob's CIDVV platform evaluates the responses to both verification calls.

7. If both verification calls receive the expected responses (486 for `+100` and 603 for `+101`) within the Validity Window, Bob's CIDVV platform:

   a. Considers the vouching successful.

   b. Allows the original call from Alice to proceed and ring through to Bob.

8. If either verification call fails to receive the correct response, times out, or does not complete within the Validity Window, Bob's CIDVV platform:

   a. Considers the vouching failed.

   b. May take any of the following actions on the original call (implementation specific):

   - Reject the call (e.g., with SIP 603 Decline).
   - Route the call to Bob's voicemail.
   - Allow the call to ring through to Bob with a visual warning on the display (e.g., "Unverified Caller-ID").

### Detailed Vetting Procedure

When Alice wants to confirm that Bob controls a particular telephone number, the following protocol is used:

1. Alice and Bob share a secret (e.g., "elephant").

2. Alice and Bob agree on the Caller-ID that Alice will use to vet Bob's number.

3. Alice's CIDVV platform initiates a "Wake Call" to Bob's number using a Calling Party Number consisting of the `101` prefix followed by Alice's agreed vetting Caller-ID.

4. Bob's CIDVV platform performs the following actions:

   a. Intercepts the incoming call.

   b. Recognizes Alice's vetting Caller-ID and identifies this as a "Wake Call".

   c. Computes a short-lived "Recognize Token" derived from Alice's number + Bob's number + the shared secret (example: 13093956974).

   d. Stores this token temporarily in memory.

   e. Rejects the call with a SIP 603 Decline response.

   f. Considers this a successful "Wake Call" from Alice to Bob (step 1 of 3).

      The Wake Call alone does not authenticate Alice and does not cause
      Bob's platform to initiate an Auth Call. Alice's vetting Caller-ID
      may be known or reused across multiple vetting relationships, so
      Bob's platform waits for a valid Recognize Call proving knowledge
      of the shared secret before placing any return call to Alice. This
      prevents an attacker from spoofing Alice's vetting Caller-ID in a
      Wake Call in order to goad Bob's platform into calling Alice.

6. Alice's CIDVV platform performs the following actions upon receiving the 603 response:

   a. Considers Bob's CIDVV platform "Awake".

   b. Independently calculates the same "Recognize Token" (example: 13093956974).

   c. Calculates an "Auth Token" using a slightly different derivation from the shared secret + Bob's number + Alice's number (example: 12674591304).

   d. Stores the Auth Token temporarily in memory.

   e. Initiates a "Recognize Call" (step 2 of 3) to Bob using the Recognize Token as the Caller-ID, prefixed with `+101` (example: `+10113093956974`).

7. Bob's CIDVV platform performs the following actions upon receiving the Recognize Call:

   a. Intercepts the call.

   b. Verifies that the received Caller-ID matches the previously stored Recognize Token.

   c. Considers this a successful "Recognize Call" from Alice to Bob (step 2 of 3).

   d. Trusts Alice's CIDVV platform based on the matching token.

   e. Rejects the call with a SIP 486 (Busy Here) response.

   f. Computes the corresponding "Auth Token" (example: 12674591304).

   g. Initiates an "Auth Call" (step 3 of 3) to Alice using the Auth Token as the Caller-ID, prefixed with `+101` (example: `+10112674591304`).

8. Alice's CIDVV platform performs the following actions upon receiving the Auth Call:

   a. Intercepts the call.

   b. Verifies that the received Caller-ID matches the previously stored Auth Token.

   c. Considers this a successful "Auth Call" from Bob to Alice (step 3 of 3).

   d. Trusts Bob's CIDVV platform based on the matching token.

   e. Rejects the call with a SIP 486 (Busy Here) response.

   f. Considers the vetting procedure complete and successful.

9. Bob's CIDVV platform receives the 486 (Busy Here) response from Alice and also considers the vetting procedure successful.

Only a party that can receive calls for Bob's number and that knows the
shared secret can complete the expected token and response sequence.

### Signaling Prefixes and Call Types

CIDVV uses the following special prefixes in the Calling Party Number:

| Prefix | Call Type          | Direction      | Purpose                                       | Expected Response (by callee) |
|--------|--------------------|----------------|-----------------------------------------------|-------------------------------|
| +100   | Vouch Phase 1      | Bob -> Alice   | Vouching verification (Phase 1)               | 486 Busy Here                 |
| +101   | Vouch Phase 2      | Bob -> Alice   | Vouching verification (Phase 2)               | 603 Decline                   |
| +101   | Wake Call          | Alice -> Bob   | Initiate vetting and trigger token generation | 603 Decline                   |
| +101   | Recognize Call     | Alice -> Bob   | Prove knowledge of shared secret              | 486 Busy Here                 |
| +101   | Auth Call          | Bob -> Alice   | Prove control of destination number           | 486 Busy Here                 |

**Note:** All vetting-related calls use the `+101` prefix. Context is determined by the Caller-ID used (vetting Caller-ID vs. token value) and the current state maintained by the CIDVV platform.

## Response Semantics

Because intermediate SIP and SS7/TDM networks may translate,
modify, or replace response codes, implementations MUST interpret
responses based on behavioral class (e.g., "Busy"-class vs.
"Rejection"-class) rather than exact numeric values.

Implementations SHOULD use SIP 486 (Busy Here) and 603 (Decline)
as the canonical representations of these behaviors where possible.

For calls with the "101" prefix, a CIDVV platform normally returns a
Rejection-class response such as SIP 603 Decline. The platform returns a
Busy-class response such as SIP 486 Busy Here only when the "101" call
matches an active vetting token-confirmation step.

CIDVV requires that these response behaviors remain distinguishable
across the signaling path. Environments that cannot preserve this
distinction may not support CIDVV vouching.

### Phase 1 Verification ("100" Prefix)

A call using the "100" prefix is the **Phase 1** verification call. It succeeds only if it receives a "Busy"-class response (e.g., SIP 486 Busy Here).

### Phase 2 Verification ("101" Prefix)

A call using the "101" prefix is the **Phase 2** verification call. It succeeds only if it receives a "Rejection"-class response, with SIP 603 Decline as the canonical example.

### Combined Phase Behavior (Required for Vouch Success)

A successful vouch requires both Phase 1 and Phase 2 to complete with
their expected behaviors within the Validity Window.

Implementations MUST NOT treat a single phase as sufficient for a
successful vouch. If either phase fails, is missing, altered, delayed, or
inconsistent, the vouch result MUST be treated as unsuccessful or
indeterminate.

Vetting success is defined separately by the Wake, Recognize, and Auth
procedure in Section [Vetting Procedure](#vetting-procedure).

# Protocol Operation

## Vouching Procedure

In this procedure, Alice places a call to Bob using an Asserted
Caller-ID. Bob's CIDVV platform uses the vouching procedure to determine
whether the party responsible for Alice's Asserted Caller-ID will vouch
for this specific call attempt. The result of the vouching procedure is
a verification signal; handling of the original call is
implementation-specific.

The following subsections define the required behavior for vouching.

Alice initiates a call to Bob using her Asserted Caller-ID

Alice's CIDVV platform receives a notification of an attempted call from
Alice to Bob using Alice's Asserted Caller-ID. The notification mechanism
is implementation specific. For example, Alice's SBC or gateway might
send an INVITE to the CIDVV platform before advancing the original call
toward the PSTN.

Upon receiving this notification, Alice's CIDVV platform MUST cache the
attempted call using the tuple:

   (Asserted Caller-ID, Called Number)

for the Validity Window.

The response used for this local notification is outside the
inter-domain CIDVV verification exchange. Implementations MAY use any
local behavior that allows Alice's SBC or gateway to continue routing the
original call toward Bob.

When Bob's CIDVV platform receives the original call, it holds the call
and initiates two short signaling-only verification calls toward Alice's
Asserted Caller-ID. These calls MAY be performed in either order or in
parallel.

### Phase 1 Verification ("100")

Bob's CIDVV platform constructs a verification Calling Party Number by
prefixing "100" to the rightmost 12 digits of Bob's called number after
number normalization. It then initiates a verification call toward
Alice's Asserted Caller-ID using that value as the Calling Party Number.

When Alice's SBC receives a call with a Calling Party Number beginning
with "100", it MUST route the call to Alice's CIDVV platform.

Upon receiving the Phase 1 verification call, Alice's CIDVV platform
MUST determine whether the call matches cached state for the Asserted
Caller-ID that received the verification call and the called-number value
encoded in the verification Calling Party Number.

If a matching cache entry exists within the Validity Window, Alice's
CIDVV platform MUST reject the verification call with a Busy-class
response, such as SIP 486 Busy Here.

If no matching cache entry exists, Alice's CIDVV platform MUST NOT return
a Busy-class response for the Phase 1 verification call. It MAY reject
the call with a Rejection-class response, such as SIP 603 Decline.

### Phase 2 Verification ("101")

Bob's CIDVV-aware element initiates a second verification call using a
Calling Party Number constructed by prefixing "101" to the same
12-digit payload used for Phase 1.

When Alice's SBC receives a call with a Calling Party Number beginning
with "101", it MUST route the call to Alice's CIDVV platform.

For vouching purposes, Alice's CIDVV platform does not need to perform a
vouching cache lookup for the Phase 2 call. Unless the call corresponds
to an active vetting procedure, Alice's CIDVV platform MUST reject the
call with a Rejection-class response, such as SIP 603 Decline.

A Phase 2 verification call does not, by itself, prove that Alice has an
active call in progress to Bob. Phase 2 is used together with Phase 1 to
distinguish a CIDVV-aware platform from ordinary network behavior and to
reduce false-positive vouches.

A "101" call that corresponds to an active vetting procedure is handled
according to Section [Vetting Procedure](#vetting-procedure).

### Combined Phase Behavior (Required for Vouch Success)

A successful vouch requires both Phase 1 and Phase 2 to complete with
their expected behaviors within the Validity Window.

Implementations MUST NOT treat a single phase as sufficient for a
successful vouch. If either phase fails, is missing, altered, delayed, or
inconsistent, the vouch result MUST be treated as unsuccessful or
indeterminate.

Vetting success is defined separately by the Wake, Recognize, and Auth
procedure in Section [Vetting Procedure](#vetting-procedure).

### Vouch Call Timers

The Validity Window controls how long cached vouching state remains
valid at the originating CIDVV platform.

Independently, the platform that initiates a Phase 1 or Phase 2
verification call SHOULD implement a configurable local timer that
controls how long it waits for a signaling response to that verification
call.

A default timeout of 3-6 seconds is reasonable for domestic calls.
For international destinations, longer timeouts (typically 8-20 seconds)
are recommended to accommodate higher Post-Dial Delay (PDD).

The two vouch calls (Phase 1 and Phase 2) may be initiated sequentially
or simultaneously.

## Correlation Model

CIDVV vouching correlates calls using the Asserted Caller-ID, the called
number, and a Validity Window. It does not attempt to identify individual
call legs across the PSTN.

A successful vouch indicates that at least one matching call attempt
occurred during the Validity Window. It does not prove a one-to-one
correspondence between a specific original call leg and a specific
verification call.

When multiple calls with the same Asserted Caller-ID and called number
occur within the Validity Window, implementations MAY treat the tuple as
active state rather than requiring strict one-to-one correlation. This
case is discussed further in Section [Multiple Simultaneous Calls from
the Same Caller-ID](#fan-out).

## State Storage and Multi-Tenant Isolation
{: #state-storage }

CIDVV implementations maintain short-lived state for vouching and
vetting. The representation of that state is implementation specific and
is not carried on the wire.

For vouching, implementations commonly store state associated with the
Asserted Caller-ID, the called number, and the Validity Window. For
vetting, implementations store temporary state associated with the Wake,
Recognize, and Auth steps, including any pending Recognize Token or Auth
Token.

Implementations MAY use hashes, derived keys, database keys, in-memory
objects, or other local mechanisms for state storage. These internal
storage keys MUST NOT alter the externally visible token computation
defined in Section [Token Computation Algorithm](#token-computation).

All temporary state MUST expire automatically. Loss of state, expiration
of state, or inability to retrieve state MUST cause the corresponding
vouching or vetting operation to fail closed.

CIDVV platforms that operate on behalf of multiple independent customers
MUST ensure that all vouching and vetting state is scoped per customer
or tenant. This prevents unrelated customers from interacting through
shared state, identical telephone-number tuples, identical token values,
or misconfigured shared secrets.

Implementations MAY use separate storage, partitioning, customer-specific
configuration, tenant identifiers, or access-control boundaries to
achieve this isolation.

## Vetting Procedure
{: #vetting-procedure }

Vetting a remote number requires three separate calls (distinct SIP
dialogs) using a pre-agreed shared secret. The process confirms that
the **called party (Bob)** controls the target telephone number and
possesses the correct shared secret. In the examples below, Alice is
the verifier who initiates the vetting procedure to Bob's number in order to
vet Bob's number.

Before vetting begins, Alice and Bob agree on a shared secret, Alice's
vetting Caller-ID, and a Validity Window.

Alice places a Wake Call to Bob using a Calling Party Number consisting
of the "101" prefix followed by Alice's agreed vetting Caller-ID.

When Bob's CIDVV platform receives the Wake Call, it removes the "101"
prefix and verifies that the resulting Caller-ID is expected for the
current vetting attempt.

Bob's platform MUST compute the Recognize Token using the algorithm
defined in Section [Token Computation Algorithm](#token-computation) and
store the resulting token for the Validity Window. It then rejects the
Wake Call with SIP response 603 (Decline).

Alice computes the same Recognize Token and places a Recognize Call to
Bob using a Calling Party Number consisting of the "101" prefix followed
by the Recognize Token.

When Bob's CIDVV platform receives the Recognize Call, it removes the
"101" prefix and compares the remaining numeric value to the recently
cached Recognize Token.

If the Recognize Token matches, Bob's CIDVV platform MUST reject the
Recognize Call with SIP response 486 (Busy Here). Alice's platform treats
this response as a successful Recognize.

Bob's CIDVV platform then computes the Auth Token using the algorithm
defined in Section [Token Computation Algorithm](#token-computation) and
places an Auth Call to Alice using a Calling Party Number consisting of
the "101" prefix followed by the Auth Token.

When Alice's CIDVV platform receives the Auth Call, it removes the "101"
prefix and compares the remaining numeric value to the expected Auth
Token.

If the Auth Token matches, Alice's CIDVV platform MUST reject the Auth
Call with SIP response 486 (Busy Here). Bob's platform treats this
response as a successful Auth.

Any other response, timeout, token mismatch, expired cache entry, or
unexpected Caller-ID MUST be treated as an unsuccessful vet.

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
 - "404" is only an example of how Alice's CIDVV platform might respond to a local notification from Alice's SBC or gateway. The response used for this local notification is implementation-specific and is not part of the inter-domain CIDVV verification exchange.
 - "+100" represents a verification call whose Calling Party Number begins with the prefix "100" (or "+100") followed by Bob's called number
 - "+101" represents a verification call whose Calling Party Number begins with the prefix "101" (or "+101") followed by Bob's called number
 - "302" is only an example of call advancement after a successful vouch. CIDVV does not require use of SIP 302; implementations may use any local method to continue, redirect, or otherwise handle the original call.

### Successful Vouch Step-by-Step Description

The diagram above shows the high-level message flow. The following numbered steps provide the detailed behavior, including Caller-ID manipulation performed by CIDVV platforms.

Note that the two verification calls (Phase 1 and Phase 2) MAY be performed in either order or in parallel.

1. The originating user (Alice, Asserted Caller-ID `+12125550100`) initiates a call to Bob (`+19495550199`).

2. The call is routed from Alice's User Agent to her SBC, which forwards it to the originating CIDVV platform (CIDVV_A).

3. **CIDVV_A**:
   - Caches the call attempt using the tuple `(Calling: 12125550100, Called: 19495550199)` for the Validity Window.
   - Releases the local notification leg using implementation-specific behavior, such as a SIP final response, so that the original call can continue toward Bob.

4. After the local notification leg is released, Alice's SBC or gateway
   continues routing the original call toward the PSTN using Alice's
   original Caller-ID.

6. The call reaches Bob's SBC via the PSTN. Because this is a new dialog (initial INVITE), Bob's **CIDVV-aware SBC** forwards the call to the terminating CIDVV platform (CIDVV_B).

7. **CIDVV_B** initiates Phase 1 and/or Phase 2 verification calls toward Alice's number (`+12125550100`):
   - Phase 1: Caller-ID = `+10019495550199`
   - Phase 2: Caller-ID = `+10119495550199`

8. Each verification call arrives at Alice's SBC via the PSTN.

9. **Alice's SBC** detects the leading `+100` or `+101` prefix and routes the call to CIDVV_A.

10. **CIDVV_A**:
   - For a `+100` prefix: Looks up the cached tuple for the Validity Window and returns SIP **486 Busy Here** if matching state exists.
   - For a `+101` prefix: Returns SIP **603 Decline** for vouching purposes unless the call corresponds to an active token-confirmation step.

11. **CIDVV_B** receives the expected responses for both phases. Once both Phase 1 and Phase 2 have succeeded, it considers the combined result a **Successful Vouch**.

12. **CIDVV_B** determines the vouch result and applies its local call-handling policy. If the call is to be advanced, CIDVV_B may release or redirect the held call using implementation-specific behavior, such as returning SIP **302 Moved Temporarily** to Bob's SBC.

13. Bob's SBC or serving network element advances the original call according to that local behavior, for example by routing the call to Bob's User Agent.

14. Bob's telephone rings.

This mechanism allows Bob's CIDVV platform to verify Alice's Asserted
Caller-ID before deciding how to handle the original call.

## Unsuccessful Vouch

A successful vouch requires both verification phases to complete with the expected response behavior within the Validity Window.

If either phase fails, times out, produces an unexpected response, or cannot be completed, the vouch MUST be treated as unsuccessful or indeterminate. An implementation MAY stop the verification procedure early after any phase has already failed.

For example:

- If the Phase 1 verification call using the `"100"` prefix does not result in a Busy-class response, such as SIP 486 Busy Here, the vouch MUST fail. The implementation MAY omit the Phase 2 verification call.
- If the Phase 1 verification succeeds but the Phase 2 verification call using the `"101"` prefix does not result in a Rejection-class response, such as SIP 603 Decline, the vouch MUST fail.

Implementations MUST fail closed. Any ambiguity, unexpected response, timeout, or call progression MUST result in an unsuccessful or indeterminate outcome.

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
{: title="Recognize Call (101 prefix) - Alice proves knowledge of the shared secret using the Recognize Token"}

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
{: title="Auth Call (101 prefix) - Bob proves knowledge of the shared secret using the Auth Token"}

### Successful Caller-ID Vetting Flow

Vetting a remote number requires three separate calls (distinct SIP dialogs) that together form a single challenge-response operation protected by a pre-agreed shared secret. The process confirms that the called party (Bob) controls the target telephone number and possesses the correct shared secret.

1. Alice and Bob agree on a shared secret (e.g., `elephant`) and Alice's vetting Caller-ID (e.g., `+12125550100`).

2. Both parties configure their CIDVV platforms with the shared secret, Alice's vetting Caller-ID, and an optional authorization lifetime (e.g., "one week"). This allows the number owner (Bob) to limit how long the shared secret remains active for vetting.

3. Alice's CIDVV platform (CIDVV_A) initiates the **Wake Call** to Bob's number (`+19495550199`) with Caller-ID `+10112125550100`.

4. Bob's SBC, gateway, or other CIDVV-aware network element recognizes
   the `101` prefix in the Calling Party Number and routes the call to
   CIDVV_B.

6. **CIDVV_B**:
   - Strips the `101` prefix to recover Alice's vetting Caller-ID.
   - Recognizes it as a pre-agreed vetting number.
   - Computes the Recognize Token.
   - Caches the token for the Validity Window.
   - Rejects the call with SIP **603 Decline**.

7. CIDVV_A receives the 603 and computes the same Recognize Token.

8. CIDVV_A immediately places the **Recognize Call** to Bob using Caller-ID `+101` followed by the Recognize Token (e.g., `+10113093956974`).

9. Bob's SBC, gateway, or other CIDVV-aware network element recognizes
   the `101` prefix in the Calling Party Number and routes the call to
   CIDVV_B.

10. **CIDVV_B**:
   - Strips the `101` prefix.
   - Verifies the token matches the cached value.
   - Responds with SIP **486 Busy Here**.

11. CIDVV_A receives the 486 and computes the Auth Token.

12. **CIDVV_B**:
    - Computes the Auth Token.
    - Initiates the **Auth Call** to Alice's vetting Caller-ID using
      Caller-ID `+101` followed by the Auth Token
      (e.g., `+10112674591304`).

13. Alice's SBC, gateway, or other CIDVV-aware network element recognizes
    the `101` prefix in the Calling Party Number and routes the call to
    CIDVV_A.

14. **CIDVV_A**:
    - Strips the `101` prefix.
    - Verifies the Auth Token.
    - Responds with SIP **486 Busy Here**.
    - Declares the vetting successful.

15. CIDVV_B receives the 486 and also declares the vetting successful.

All calls are short signaling-only exchanges. The entire operation MUST complete within the Validity Window.

#### Multi-Number Vetting Optimization (Multivetting)

To reduce signaling load when vetting multiple numbers controlled by the same party (Bob), a single Wake call MAY establish shared state. Subsequent Recognize/Auth pairs for additional numbers can then proceed, with the Validity Window refreshed after each successful individual vet.

In this optimization:

1. Alice performs **one Wake Call** to any of Bob's numbers (or a designated primary vetting number) using her pre-agreed vetting Caller-ID prefixed with `+101`. Bob's CIDVV_B creates and caches a session context (tied to Alice's vetting Caller-ID + shared secret).

2. For each number to vet (including the first if desired):
   - Alice sends the **Recognize Call** using the per-number Recognize Token.
   - **CIDVV_B** verifies the token against the current call (using calling number, called number and the shared secret) and responds with 486 Busy Here if successful.
   - The per-number **Auth Call** proceeds as in the base flow.
   - Upon successful completion of each individual vet (mutual 486 acknowledgments), both sides refresh/restart the Validity Window for the ongoing session.

Implementations SHOULD enforce a configurable maximum session lifetime
for a multivetting operation, independent of per-vet Validity Window
refreshes.

This optimization is **OPTIONAL**. Implementations MUST support the base three-call-per-number sequence. Multivetting requires Bob's platform to maintain per-Alice session state. The Wake call establishes "Bob is awake and shares the secret"; Recognize/Auth pairs confirm per-number control.

All other requirements, including token computation as defined in Section
[Token Computation Algorithm](#token-computation), per-vet failure
handling, and state isolation as described in Section
[State Storage and Multi-Tenant Isolation](#state-storage), remain
unchanged.

**Rationale**: This balances efficiency for bulk vetting (e.g., enterprise portfolios or service-provider batch validation) with strong per-number security properties and operational flexibility.

#### Vetting Failure Cases

A vetting attempt may fail for the following reasons:

A vetting attempt may fail for the following reasons:

* Bob does not have a participating CIDVV platform; the Wake Call may not
  produce the expected 603 response, the Recognize Call may not produce
  the expected 486 response, and Bob will not initiate a valid Auth Call
  back to Alice.

* The shared secret, Alice's vetting Caller-ID, or Validity Window does
  not match; the three-call sequence will not produce the expected
  603 + 486 + 486 behavior.

* Network or policy restrictions prevent one or more calls from reaching
  the remote CIDVV platform.

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

Networks and SBCs that recognize CIDVV signaling SHOULD prevent calls
with Calling Party Numbers beginning with "100", "101", "+100", or
"+101" from being presented to ordinary end users.

When a responsible CIDVV platform is available, such calls SHOULD be
routed to that platform for protocol handling. Intermediate networks
SHOULD NOT generate successful CIDVV responses on behalf of a CIDVV
platform unless explicitly configured to do so by the responsible
administrative domain.

CIDVV signaling calls SHOULD result in a non-success response, commonly
486 Busy Here or 603 Decline, and MUST NOT establish media.

Call analytics, labeling, and fraud detection systems SHOULD recognize
these prefixes and treat the calls as protocol signaling rather than
ordinary subscriber traffic.

## Carrier Incentives and SBC Policies

CIDVV signaling calls are short-lived and do not require media exchange.
They represent a small incremental signaling load compared with the
network cost of fraudulent, spoofed, and nuisance calling.

Carriers and service providers benefit when call participants can produce
stronger evidence of Caller-ID control. Such signals can support fraud
analytics, traceback, branded-calling programs, customer reputation
systems, and local call-handling policy without requiring intermediate
carriers to make a final identity determination for every call.

Within the responsible administrative domain for a destination number, an
SBC, proxy, or other CIDVV-aware network element SHOULD recognize calls
where the Calling Party Number begins with `+100`, `+101`, `100`, or
`101` and route such calls to the appropriate CIDVV platform when one is
available.

If no responsible CIDVV platform is available, the call SHOULD NOT be
presented to an ordinary end user.

Intermediate networks SHOULD NOT block, rewrite, or specially rate-limit
CIDVV signaling calls solely because the Calling Party Number contains a
CIDVV prefix.

Intermediate networks SHOULD NOT synthesize responses that would be
interpreted as successful CIDVV behavior unless explicitly configured to
do so by the responsible administrative domain.

CIDVV signaling calls MUST NOT establish media. If a CIDVV signaling call
would otherwise result in media establishment, the responsible network
element SHOULD terminate the attempt with a non-success response in a way
that does not create a false-positive CIDVV verification result.

## Response Variability

Implementations SHOULD interpret responses based on behavioral class
(e.g., "busy" class such as 486 Busy Here versus "rejection" class such
as 603 Decline) rather than relying on exact numeric
values. Intermediate networks may translate or modify response codes,
so behavioral class is the preferred signal.

## Short-Term State Management

CIDVV relies on short-lived state for the (Calling Number, Called Number)
tuple, valid only for the Validity Window. Implementations MUST expire
this state automatically after the Validity Window.

State used for CIDVV verification MUST be scoped so that unrelated calls,
customers, tenants, or administrative domains cannot satisfy each other's
verification requests.

## International and Cross-Border Operation

International calls often experience higher Post-Dial Delay. Vouching
call timeouts SHOULD be adaptive: 3-6 seconds is appropriate for domestic
calls, while 8-20 seconds or more may be needed for international
destinations based on observed PDD.

Implementations SHOULD choose verification-call timing and retry behavior
based on observed network conditions, especially on international routes.

# Operational Considerations

## Protocol Operation - Vouching

If a vouching call results in a provisional response (e.g., 180
Ringing) or a successful response (200 OK), the platform that initiated
the verification call SHOULD immediately cancel the call and treat the
remote system as not implementing CIDVV.

## Failure and Restart Behavior

CIDVV platforms rely on short-lived state. Upon restart or loss of
state, implementations SHOULD continue accepting new call deposits, but
MUST treat verification requests as unsuccessful until fresh matching
state has been deposited.

Implementations SHOULD fail closed rather than risk false-positive
validation. When verification cannot be performed, implementations SHOULD
return a non-success response, such as a 4xx, 5xx, or 6xx response. SIP
603 Decline is commonly used for this purpose.

## Number Normalization
{: #number-normalization }

All telephone numbers used in CIDVV operations (for caching, token
generation, comparison, etc.) MUST be normalized to a plain digit string
in E.164 format **without any leading "+" sign**.

CIDVV encodes signaling information in the Calling Party Number. Because
some PSTN and ISDN signaling environments limit the Calling Party Number
to 15 digits, CIDVV values that would exceed this limit are truncated as
specified below.

1. Remove any leading "+" or other punctuation characters (parentheses,
   dashes, spaces, etc.).

2. Use the full E.164 representation: country code followed by the
   national significant number.

3. No padding is performed. If truncation is required to stay within the
   15-digit limit for the Calling Party Number field, always remove
   leading digits, preserving the rightmost digits.

The leading `+` sign may be present in SIP signaling or user-facing
displays. It MUST be stripped before any CIDVV processing, tuple storage,
comparison, or token computation.

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
{: #fan-out }

In some scenarios, multiple calls may legitimately originate from the
same Asserted Caller-ID to the same called number within a short period
of time (e.g., an office full of people calling a radio contest to be
the "100th caller", a call center, or a political phone bank).

In such cases, the originating CIDVV platform **MAY** switch to
"multi-call" mode for that (Asserted Caller-ID, Called Number) tuple:

- Instead of enforcing strict one-to-one correlation, the platform treats
  the tuple as active for the Validity Window.
- Each new outbound call for that tuple restarts or extends the
  expiration timer.
- The platform keeps the tuple active for the Validity Window, allowing
  multiple parallel or closely spaced calls to be vouched successfully.

This "multi-call" mode is an implementation-specific optimization.
Platforms that do not support it MAY reject, rate-limit, or treat as
indeterminate excessive concurrent calls from the same number.

### Forwarded, Translated, and Mapped Numbers
{: #mapped-numbers }

CIDVV operations depend on the number that the calling party attempted to
reach. In some deployments, that number is not the same number on which
the CIDVV platform ultimately receives the call. Examples include
toll-free translation, DID forwarding, call forwarding, burner numbers,
and other number-mapping services.

In these cases, the CIDVV platform receiving the call needs reliable
knowledge of the original called number, or a bounded configured set of
possible original called numbers, in order to perform CIDVV successfully.

#### Vouching Mapped Numbers

If Alice calls `18005550199` and the call is delivered to Bob's CIDVV
platform on `19495550199`, Alice's CIDVV platform will have cached the
tuple:

   (Alice's Asserted Caller-ID, `18005550199`)

In that case, Bob's CIDVV platform needs to perform the vouching
procedure using `18005550199` as the called-number value encoded in the
Phase 1 and Phase 2 verification Calling Party Numbers. A vouching
attempt using `19495550199` may fail because Alice's CIDVV platform has
no matching cached state for `19495550199`.

If Bob's CIDVV platform has reliable knowledge of the original called
number, it SHOULD use that number for vouching. If Bob's platform has a
small configured set of possible original called numbers that may have
delivered the call, it MAY attempt vouching against multiple candidate
called numbers, subject to local rate limits and timeout policy.

If the original called number is not known, and the candidate set is not
bounded or reliable, Bob's CIDVV platform SHOULD treat the vouching
result as unavailable or indeterminate rather than attempting to vouch
using an unrelated receiving number. Local policy may then allow, label,
bypass, route differently, reject, or otherwise handle the original call.

#### Vetting Mapped Numbers

Mapped or translated numbers can also affect vetting because the number
Alice attempts to vet may differ from the number on which Bob's CIDVV
platform receives the Wake Call. For example, Alice may attempt to vet
Bob's toll-free number `18005550199`, but the Wake Call may be translated
and delivered to Bob's CIDVV platform on DID `19495550199`.

Vetting tokens are computed using the number being vetted, not
necessarily the number on which the Wake Call is ultimately received. If
Bob's CIDVV platform has reliable knowledge of the original called number
`18005550199`, it SHOULD compute the Recognize Token and Auth Token using
`18005550199` as the called-number input.

If Bob's CIDVV platform receives the Wake Call on `19495550199` and has a
bounded, configured mapping from `19495550199` to one or more possible
public numbers, it MAY compute and temporarily cache candidate Recognize
Tokens and Auth Tokens for those mapped numbers.

When Alice sends the Recognize Call, Bob's CIDVV platform matches the
received token against the cached candidate Recognize Tokens. The matched
candidate identifies which called number Alice is attempting to vet. Bob
then uses the corresponding Auth Token for that same called number when
placing the Auth Call back to Alice.

If no candidate Recognize Token matches, the vetting attempt MUST fail.

Candidate-token generation for mapped numbers SHOULD be limited to a
small, configured, authoritative set of mappings. Implementations SHOULD
NOT generate tokens for an unbounded set of possible forwarded,
translated, or indirectly mapped numbers.

### Call Forwarding / Diversion

When the called party (Bob) has forwarded the call to a different
destination, the terminating CIDVV platform at the forwarded destination
may be the platform that performs the vouching procedure.

In this case, the terminating CIDVV platform SHOULD use the original
called number (Bob's number), when available from diversion or redirection
information (e.g., SIP `Diversion` header or ISDN Redirecting Number
field), as the called-number value encoded in the Calling Party Number of
the return vouch calls.

The return vouch calls are still directed to Alice's Asserted Caller-ID.
However, their Calling Party Numbers encode Bob's original called number,
for example:

- Phase 1: `+100` followed by Bob's number.
- Phase 2: `+101` followed by Bob's number.

Alice's CIDVV platform can then correlate the verification calls with the
cached state for Alice's original call to Bob. The terminating CIDVV
platform is responsible for performing the vouch on behalf of the
forwarded call destination.

# Security Considerations

CIDVV verification is probabilistic and based on reachability rather than
cryptographic identity. It is intended to complement, not replace,
mechanisms such as STIR/SHAKEN.

**Important Limitation**

CIDVV specifically addresses Caller-ID spoofing and impersonation. It
does **not** prevent all forms of telephone fraud. Callers who use
numbers they legitimately control can still obtain successful vouches,
even if the calls themselves are fraudulent, abusive, or unwanted.

Similarly, a malicious or compromised CIDVV platform, carrier, or service
provider could incorrectly vouch for calls using numbers under its
control. However, this does not allow that platform to vouch for
arbitrary third-party numbers unless it can also receive verification
calls for those numbers. The scope of the failure is therefore limited to
the numbers and routing authority controlled by that platform or
provider.

These cases are outside the scope of the spoofing protection CIDVV
provides.

CIDVV's security properties derive primarily from the difficulty a
spoofing caller faces in receiving calls at the Asserted Caller-ID, which
is the number being vouched.

CIDVV validates reachability within a short Validity Window rather than
providing strict per-call correlation. As a result, a successful vouch
indicates that at least one matching call attempt occurred within the
Validity Window, not necessarily that a specific verification call maps
one-to-one to a specific original call leg.

The use of distinct response patterns across the two verification calls
-- Phase 1 with the "100" prefix expecting a Busy-class response, and
Phase 2 with the "101" prefix expecting a Rejection-class response --
increases resistance to false-positive validation caused by common
network behaviors.

## Trust Model

CIDVV assumes that:

- The PSTN routes calls for a telephone number to the responsible service
  provider, administrative domain, or platform serving that number.

- A CIDVV platform acting for the Asserted Caller-ID can receive
  verification calls directed to that Caller-ID and route them to the
  appropriate CIDVV processing function.

- A CIDVV platform performing vouching for the called party can originate
  short return signaling calls toward the Asserted Caller-ID.

- Intermediate networks may modify signaling but will generally preserve
  sufficient information to allow correlation of requests and responses.

- CIDVV verification traffic related to attempted use of a telephone
  number as an Asserted Caller-ID is routed toward the party responsible
  for that number.

CIDVV does not assume that Caller-ID values are trustworthy; instead, it
verifies control through network reachability.

## Replay and Ride-Along Attacks

CIDVV relies on short-lived state to correlate signaling exchanges. This
significantly limits the window for replay and ride-along attacks.

**Replay Attacks**

A party that observes a successful verification exchange cannot
effectively replay it after the cached state expires. Implementations
**MUST** expire cached state quickly. A recommended default is **10
seconds**, although longer values (e.g., 15-30 seconds) **MAY** be used
for international or high-latency routes.

Implementations **MUST** reject verification attempts that do not match
recent, valid cached state.

**Ride-Along Attacks**

A ride-along attack is limited to the same correlation tuple used for
vouching: the Asserted Caller-ID, the called number, and the Validity
Window. A successful vouch for one called number does not allow a caller
to vouch calls to other destinations, and a successful vouch for one
Asserted Caller-ID does not apply to other Caller-IDs.

A spoofing caller might attempt to place an additional call using the
same Asserted Caller-ID to the same called number while matching cached
state is still valid. Because the originating CIDVV platform controls how
long cached state remains valid, implementers have flexibility in
mitigation:

- An implementation **MAY** delete or expire the cached state immediately
  after successfully processing the Phase 1 ("100") verification call and
  returning the Busy-class response.

- An implementation **MAY** keep the cached state active for the Validity
  Window to support legitimate parallel or closely spaced calls from the
  same Asserted Caller-ID to the same called number.

The choice of when to expire or delete state is left to the implementer,
as it involves a trade-off between reducing ride-along opportunities and
supporting legitimate simultaneous-call or fan-out behavior.

## Spoofing Resistance

CIDVV improves resistance to spoofing by requiring the party asserting a
Caller-ID to successfully receive and respond to a return call routed via
the PSTN. If a caller attempts to impersonate Alice's Caller-ID without
being able to receive calls for Alice's number, the verification call
will normally be routed to the party responsible for Alice's number
rather than to the impersonating caller. As a result, the impersonating
caller cannot complete the vouching process under normal routing
conditions.

## Denial of Service

CIDVV introduces additional signaling traffic, and deployments need to
account for the possibility of excessive or malicious verification
attempts. Because CIDVV vouching traffic is related to ordinary call
traffic, high call volumes may legitimately produce high verification
volumes.

CIDVV can also expose useful telemetry about telephone-number usage. For
example, if a widely spoofed enterprise number is asserted on many calls,
CIDVV verification attempts may cause the responsible enterprise or
service provider to receive a large volume of return verification calls.
This traffic can help identify spoofing activity, but it can also create
operational load on the CIDVV platform and serving network.

Denial-of-service protection is therefore a deployment responsibility
shared by CIDVV platforms, SBCs, proxies, gateways, and other responsible
network elements.

Deployments SHOULD apply appropriate ingress controls at the SBC, proxy,
or other network edge, including rate limits, source filtering, anomaly
detection, and protection against repeated invalid or abusive signaling
patterns. Such controls SHOULD be designed so that high-volume legitimate
enterprise traffic and high-volume spoofing telemetry can be observed
without overwhelming the CIDVV platform.

CIDVV platforms MUST bound resource usage for temporary state. For
example, implementations MUST limit the number of concurrent cache
entries, expire old entries automatically, and avoid retaining state
beyond the Validity Window except where explicitly required by local
policy.

CIDVV platforms SHOULD fail closed under resource exhaustion. If a
platform cannot safely allocate state or process a verification request,
it SHOULD treat the verification as unsuccessful or indeterminate rather
than returning a response that could create a false-positive vouch.

Deployments SHOULD monitor CIDVV signaling volume and error patterns,
especially repeated unsuccessful attempts, unexpected response patterns,
traffic inconsistent with normal call volume, or sudden increases in
verification traffic for particular asserted numbers.

## Amplification and Reflection

CIDVV generates return signaling calls as part of its operation. Care
MUST be taken to ensure that this behavior cannot be exploited for
amplification or reflection attacks.

A spoofed call using a widely abused Caller-ID can cause verification
traffic to be directed toward the party responsible for that Caller-ID.
This is an intended part of CIDVV's reachability model and may provide
useful telemetry, but deployments SHOULD ensure that the resulting
verification traffic remains bounded and manageable.

Implementations SHOULD initiate return verification calls only in response
to valid inbound call attempts or valid CIDVV protocol steps.

Implementations SHOULD limit outbound verification-call generation using
local policy, including rate limits, concurrency limits, and safeguards
against repeated unsuccessful attempts.

When multiple candidate called numbers are used for mapped, translated,
or forwarded numbers, implementations SHOULD keep the candidate set
bounded and authoritative. A single triggering event SHOULD NOT generate
an unbounded number of verification calls.

Implementations SHOULD avoid generating multiple outbound verification
calls for a single triggering event except where explicitly required by
CIDVV operation, such as Phase 1 and Phase 2 vouching calls or bounded
candidate attempts for mapped numbers.

## Response Code Manipulation

CIDVV does not require specific SIP response codes to be preserved
end-to-end, but it does require that distinct response behaviors
(e.g., "busy" vs. "reject") remain distinguishable.

Implementations MUST interpret responses based on behavioral class
(e.g., "Busy"-class vs. "Rejection"-class) rather than exact numeric
values.

Intermediate networks and gateways SHOULD NOT translate, synthesize, or
normalize responses in a way that causes an unsuccessful CIDVV response
to be interpreted as successful CIDVV behavior.

## Data Privacy

CIDVV uses telephone numbers and related call metadata as part of normal
signaling behavior. CIDVV may also create operational telemetry about
attempted use of telephone numbers as Asserted Caller-IDs, including
successful and unsuccessful verification attempts.

Deployments SHOULD handle CIDVV signaling records, verification logs, and
related telemetry according to their normal privacy, security, retention,
and access-control policies for call signaling data.

Implementations SHOULD minimize retention of CIDVV telemetry where
practical. Temporary state used for CIDVV verification MUST be short
lived and automatically expired.

Where operationally practical, access to CIDVV logs and telemetry SHOULD
be limited to systems and personnel responsible for operations, security,
fraud prevention, debugging, or compliance.

## Hash-Based Token Security (Vetting)

## Token and Shared-Secret Security (Vetting)

The security of the vetting mechanism depends on the shared secret and
the derived Recognize and Auth Tokens. Tokens MUST be computed as
specified in Section [Token Computation Algorithm](#token-computation).

Shared secrets used for vetting MAY be long-lived configuration values.
For example, a vetting agent and a number owner might retain the same
shared secret in order to perform periodic or recurring vetting of the
same customer numbers.

Implementations MUST protect shared secrets from disclosure. If a shared
secret is disclosed, a third party may be able to complete vetting
challenges for the vetting relationships and numbers covered by that
secret. Disclosure of a vetting shared secret does not, by itself, allow
the third party to successfully vouch for calls, because vouching also
requires reachability and valid call state for the Asserted Caller-ID.

Recognize and Auth Tokens MUST be valid only within the Validity Window
and MUST be automatically expired.

Implementations SHOULD use sufficiently long and random shared secrets.
Deployments SHOULD scope shared secrets to the appropriate customer,
tenant, number owner, or vetting relationship, and SHOULD support
rotation or revocation if a secret is suspected to be disclosed.

## Failure Modes

CIDVV implementations MUST fail closed. If verification cannot be
completed due to network errors, state loss, synchronization delay,
resource exhaustion, or unexpected responses, the result MUST be treated
as unverified or indeterminate rather than successful.

CIDVV platforms commonly rely on short-lived state with automatic
expiration. After a restart or loss of temporary state, the platform MAY
continue accepting new call notifications, but it MUST NOT successfully
verify calls until fresh matching state has been deposited. During this
recovery interval, legitimate calls may fail to receive a successful
vouch, but calls without matching state will not be falsely vouched.

Distributed CIDVV deployments may include multiple ingress points,
geographically diverse SBCs, or replicated state stores. If a verification
call reaches a CIDVV processing node before the relevant call state has
been replicated to that node, the verification MUST fail closed.

Deployments that use distributed state SHOULD ensure that call-deposit
state is available to all CIDVV nodes that may receive subsequent
verification calls within the Validity Window. Implementations MAY delay
advancing the original call briefly, use synchronous replication, use
sticky routing, or apply other local mechanisms to reduce the chance that
a valid verification call arrives before matching state is available.

## Interoperability Risks

CIDVV operates across heterogeneous networks, including SIP and SS7/TDM
environments. Intermediate systems may modify Calling Party Number
values, truncate digits, alter response codes, or otherwise change
signaling behavior.

These behaviors may cause verification to fail. Implementations MUST
treat ambiguous, altered, missing, or inconsistent signaling as
unsuccessful or indeterminate rather than allowing false-positive
validation.

Deployments SHOULD test CIDVV behavior across the specific carriers,
gateways, SBCs, and interworking paths used in their environment,
especially where Calling Party Number preservation, digit truncation, or
response-class translation may occur.

## Residual Risk

CIDVV improves resistance to Caller-ID spoofing but does not provide
absolute identity assurance. It reduces the effectiveness of spoofing by
testing reachability and response behavior for the Asserted Caller-ID
within a short Validity Window.

CIDVV does not prove the intent, legitimacy, or trustworthiness of the
caller. A caller using a number it legitimately controls may still place
fraudulent, abusive, or unwanted calls and receive successful vouches for
that number.

CIDVV should therefore be treated as one verification signal among
multiple inputs to call-handling, fraud-prevention, analytics, reputation,
or policy systems.

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
- Updated termination handling so that Busy-class and Rejection-class
  responses, with canonical SIP 486 and 603 behavior, are used for
  CIDVV verification signaling.
- Added a third call to the Vetting process to ensure Alice and Bob each trust that the other possesses the shared secret.
- Added option Multi-Number Vetting optimization
- Added this "Changes from Previous Version" appendix to support long-term document maintenance across multiple revisions.
- Improved Abstract, Introduction, and call flow.
