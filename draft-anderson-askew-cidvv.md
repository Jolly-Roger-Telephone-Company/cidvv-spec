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

Caller-ID spoofing remains a significant problem in telephony,
particularly across inter-domain and international call paths where
identity frameworks may not be consistently applied.

This document defines Caller-ID Vouching and Vetting (CIDVV), a
lightweight verification mechanism that uses short-lived signaling
exchanges encoded within the Calling Party Number to confirm that a
calling party can receive calls at the Asserted Caller-ID.

CIDVV is designed to operate across heterogeneous SIP and SS7/TDM
networks without requiring new protocol extensions or persistent
identity infrastructure. It relies on existing call routing behavior
and intentionally leverages failure responses as a signaling mechanism,
using failed call attempts as evidence of number control rather than
successful call completion.

The mechanism improves resistance to Caller-ID spoofing by requiring
demonstrable control of the Asserted Caller-ID, while remaining
incrementally deployable and tolerant of intermediate network
modification.

--- middle

# Introduction

Caller-ID spoofing is widely used in fraudulent and nuisance calling,
particularly in environments where calls traverse multiple
administrative domains and heterogeneous network technologies.

This document defines Caller-ID Vouching and Vetting (CIDVV), a
mechanism that verifies caller identity through network reachability
rather than asserted identity. CIDVV requires that a party asserting a
Caller-ID be able to receive a return call at that number within a brief "Validity Window".

CIDVV operates by encoding signaling information within the Calling
Party Number and leveraging existing call routing behavior to perform
a challenge-response exchange. The protocol does not require new SIP
headers, protocol extensions, or changes to SS7 signaling, and is
designed to function across mixed SIP and TDM networks.

CIDVV is incrementally deployable and does not require universal
adoption to provide benefit. It tolerates modification of signaling
by intermediate networks and relies on signaling calls and distinct
failure-response behaviors traversing the network path.

CIDVV provides strong, real-time evidence of Caller-ID validity by
requiring that a party asserting a Caller-ID be able to receive and
respond to a return call at that number within a brief Validity Window.
While it does not provide absolute identity assurance, it offers a
practical and robust signal of trust in the presented identity.

CIDVV leverages two key elements of the existing telephone ecosystem:

* Existing routing databases and numbering plans, which provide
  authoritative routing ownership for telephone numbers.
* Digit sequences chosen to minimize conflict with valid numbering plans (e.g., "100" and "101").

The mechanism operates entirely within standard PSTN routing behavior and requires no media exchange.

# Terminology

* **Caller-ID**: The telephone number presented to the called party (what the end user sees).
* **Asserted Caller-ID**: The Caller-ID value that is being vouched or vetted by this protocol. This is the number whose control the calling party claims, and it is the value used for the CIDVV Token, state management, and correlation.
* **Calling Party Number**: The value carried in the signaling protocol (e.g., SIP `From` header or ISUP Calling Party Number parameter). In many deployments this is the same as the presented Caller-ID, but they are not always identical.
* **Alice**: The calling party and verifier. In vouching flows Alice asserts a number; in vetting flows Alice verifies Bob's number.
* **Bob**: The called party. In vetting flows Bob is the owner whose number is being vetted.
* **Mallory**: An attacker attempting to spoof a Caller-ID.
* **CIDVV Platform**: A system that implements the vouching and vetting procedures defined in this document.
* **CIDVV-aware Network Element**: An SBC or intermediary that recognizes CIDVV signaling prefixes and interprets associated responses, but does not implement the full CIDVV platform logic.
* **Vouch**: The act of a CIDVV platform asserting that it has verified control of a telephone number through the challenge-response mechanism described in this document, which may consist of one or more verification calls. A successful vouch provides strong evidence that the calling party controls the Asserted Caller-ID.
* **Vet** (or **Vetting**): The process by which a CIDVV platform confirms the relevant party controls the Asserted Caller-ID via the three-call challenge-response sequence. Vetting may be performed on behalf of third parties such as Caller-ID branding services, Vetting Agents, law enforcement agencies, trade organizations, or enterprise trust programs.
* **Vouching Call**: A short signaling call used in the CIDVV protocol. CIDVV defines **Phase 1** ("100" prefix) and **Phase 2** ("101" prefix) verification calls.
* **Phase 1 Vouch** ("100" prefix): The initial Vouch verification step. Expected response behavior is a "Busy"-class response (e.g., SIP 486 Busy Here).
* **Phase 2 Vouch** ("101" prefix): The secondary Vouch step. Expected response behavior is a "Rejection"-class response (e.g., SIP 603 Decline).
* **Successful Vouch**: Requires **both Phase 1 and Phase 2** to complete with the expected behaviors within the Validity Window.
* **Unsuccessful Vouch**: A verification result indicating that no matching cache entry was found.
* **Verification Not Performed**: A condition where verification could not be completed due to system or network conditions.
* **Validity Window**: The time interval during which the originating CIDVV platform will accept and correlate a vouch attempt (return call) from the called party. This is typically on the order of 10–30 seconds.
* **Vouch-Call Timeout**: The local timer used by the originating CIDVV platform to limit how long it will wait for a response to an individual Phase 1 or Phase 2 vouching call. This is typically 3–6 seconds for domestic calls and longer (e.g. 8–20 seconds) for international calls. It is distinct from the Validity Window.

Once successfully vouched, an Asserted Caller-ID may be referred to informally as a "vouched number".

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14, RFC 2119 and RFC 8174 when, and only when, they appear in all
capitals, as shown here.

## Motivation and Advantages

The CIDVV vouching and vetting mechanism is designed to operate with minimal new infrastructure while providing strong protection against Caller-ID spoofing. Its primary advantages are:

* **Leverages existing PSTN infrastructure**: Uses existing numbering plans and routing databases without requiring new infrastructure or protocol extensions.
* **Strong anti-spoofing protection**: A successful vouch provides strong evidence of control over the Asserted Caller-ID, because only the legitimate owner can complete the challenge-response sequence. Spoofed calls are typically rejected early.
* **Visibility into spoofing activity**: Number owners gain direct insight into how often and to where their numbers are being spoofed through logged vetting attempts.
* **Low signaling overhead**: The short vouching calls replace what would otherwise be completed fraudulent calls, resulting in lower overall network load.
* **Full TDM/SS7 compatibility**: Works natively across legacy SS7 and ISDN networks. SIP is not required.
* **International applicability**: Functions effectively across national boundaries without relying on country-specific frameworks.
* **Deployment flexibility**: Enterprises can run their own CIDVV platform using open-source tools (e.g., Kamailio or Asterisk). Service providers or third-party Vetting Agents can offer cloud-based vouching and vetting services on behalf of carriers, enterprises, or number owners.
* **Promotes competition**: Customers using third-party cloud-based CIDVV services can more easily switch between providers, fostering competition and helping to drive down costs.

This design lowers the barrier to entry and encourages broad adoption while avoiding the single points of failure and high coordination costs of centralized solutions. CIDVV is particularly suitable for service providers, enterprises, and end users who need robust Caller-ID validation using only existing telephone infrastructure.

# Design Principles

CIDVV is designed to operate under the assumption that intermediate
networks may normalize, truncate, or otherwise modify signaling
information. The protocol therefore encodes all required signaling in
a numeric Calling Party Number that can survive traversal of mixed
SIP and SS7/TDM networks.

CIDVV relies on the ability to distinguish between classes of
call rejection behavior (e.g., "busy" vs. "rejected"), rather than
requiring specific numeric response codes to be preserved end-to-end.

# Number Normalization
{: #number-normalization }

All telephone numbers used in CIDVV operations MUST be normalized to a
plain digit string in E.164 format (without the leading "+" sign) as
follows:

1. Remove any leading "+" or other punctuation characters.
2. Use the full E.164 representation: country code followed by the national significant number.
3. No padding is performed. If truncation is required to stay within the 15-digit limit, always remove leading digits (preserving the rightmost digits).

## Simple Overview
{: #simple-overview }

CIDVV is a challenge-response mechanism that proves control of an Asserted
Caller-ID using short signaling-only calls. No media establishment or new
protocol extensions are required.

### Simple Overview (Vouching)

When Alice wants to call Bob using her asserted caller-id:

1. Alice places a normal call to Bob using her Asserted Caller-ID.

2. Bob's CIDVV platform intercepts the incoming call.

3. Before ringing Bob's phone, Bob's platform initiates two short verification
   calls back to Alice:

   a. It uses Alice's Asserted Caller-ID as the destination number.

   b. It sets a special prefix in the Calling Party Number ("100" for Phase 1
      and "101" for Phase 2).

4. Alice's CIDVV platform intercepts the verification calls and responds
   with the expected behavior to prove that she controls the number and has
   an active call in progress to Bob.

5. If both verification steps succeed within the Validity Window, Bob's
   CIDVV platform allows the original call to ring through to Bob.

The two verification calls from Bob's CIDVV platform use **reachability**
to ensure that Alice really controls the Asserted Caller-ID she is presenting,
and is attempting a call to Bob's number.

### Simple Overview (Vetting)

When Alice wants to confirm that Bob controls a particular telephone number:

1. Alice and Bob share a secret (e.g., "elephant").

2. Alice initiates a "Wake Call" to Bob using a special prefix (`+101`)
   and her agreed vetting caller-id.

3. Bob's CIDVV platform recognizes the vetting caller-id, computes a short-lived
   Recognize Token, and rejects the call.

4. Alice's platform then sends a "Recognize Call" using the Recognize Token
   as the caller-id.

5. Bob's platform verifies the token and responds with an "Auth Call" using
   an Auth Token.

6. Alice's platform verifies the Auth Token. Both sides now consider the
   vetting successful.

## CIDVV Mechanisms

### Vouching Mechanism

CIDVV uses two distinct signaling prefixes in the Calling Party Number for
vouching:

* **"100"** — Phase 1 Verification Call
* **"101"** — Phase 2 Verification Call

A successful **Vouch** requires **both** Phase 1 and Phase 2 to complete
with their expected responses within the Validity Window. The two phases
MAY be performed in any order or in parallel.

Expected behaviors for vouching:

* **Phase 1** ("100" prefix): MUST receive a Busy-class response
  (e.g., SIP 486 Busy Here).
* **Phase 2** ("101" prefix): MUST receive a Rejection-class response
  (e.g., SIP 603 Call Rejected).

If either phase fails to produce the expected response within the
vouch-call timeout (or is missing, altered, or inconsistent), the entire
vouch MUST be treated as unsuccessful or indeterminate.

### Vetting Mechanism

CIDVV uses a three-step handshake for vetting. All calls use a `+101` prefix on the Calling Party Number.

- **Wake Call** (Alice → Bob): Alice initiates using her vetting caller-id.
- **Recognize Call** (Alice → Bob): Alice uses the Recognize Token as the caller-id.
- **Auth Call** (Bob → Alice): Bob uses the Auth Token as the caller-id.

A successful **Vet** requires all three steps to complete successfully within the Validity Window.

The **Recognize Call** serves as an important anti-goading protection. It ensures that Bob will only initiate the final Auth Call back to Alice if he has recently received a valid Wake Call and the Recognize Token matches what he computed. This prevents an attacker from tricking Bob’s platform into calling Alice (e.g., a script-kiddie attack).

Both sides independently compute the short-lived Recognize Token and Auth Token from the shared secret and the two telephone numbers involved. The tokens are valid only within the Validity Window.

## Hash Function for Vetting and Token Generation
{: #hash-function }

The same deterministic algorithm MUST be used by both parties to generate the Recognize Token and Auth Token, and for any short-term cache lookups.

**Algorithm (normative)**

1. Normalize both telephone numbers to E.164 digit strings (no leading "+", no punctuation) as defined in Section [Number Normalization].

2. For the **Recognize Token**:
   - Concatenate as UTF-8 bytes: `normalized-calling-number || "|" || normalized-called-number || "|" || shared-secret`

3. For the **Auth Token**:
   - Concatenate as UTF-8 bytes: `shared-secret || "|" || normalized-called-number || "|" || normalized-calling-number`

4. Compute the SHA-256 digest of the concatenated bytes.

5. Take the first 8 hexadecimal characters of the digest.

6. Convert that 8-hex string to a decimal integer.

7. Left-pad with zeros to 10 digits if needed, then prepend '1' to produce an 11-digit token.

**Example (for illustration only)**

- Calling number: `+12125550100`
- Called number: `+19495550199`
- Shared secret: `hamburger`

Recognize Token calculation:
- Concatenated string: `12125550100|19495550199|hamburger`
- SHA-256 (first 8 hex chars) → decimal conversion → padded/prepended → `12953388433`

Auth Token calculation uses the reversed order of the numbers after the secret.

Implementations MUST use identical normalization, concatenation order, and processing for both ends of the vetting exchange. Tokens are valid only inside the Validity Window.

### Detailed Vouching Procedure

When Alice wants to place a call to Bob using her asserted caller-id, the following steps are performed:

1. Alice’s CIDVV platform is notified of her outbound call attempt to Bob (using her Asserted Caller-ID). This notification is typically done by Alice’s SBC or gateway sending an INVITE to the CIDVV platform. The CIDVV platform rejects this INVITE with a SIP 404 (Not Found) response so that the original call can continue toward the PSTN. Note that the exact notification mechanism is implementation-specific and may use other methods (e.g., an API call or STIR signing request) in future deployments.

2. Bob’s CIDVV platform intercepts the incoming call from Alice and holds it (does not yet alert Bob’s phone).

3. Bob’s CIDVV platform initiates two short verification calls back to Alice (in either order or in parallel):

   a. One verification call with Calling Party Number prefixed by `+100`.

   b. One verification call with Calling Party Number prefixed by `+101`.

   Both calls are directed to Alice’s Asserted Caller-ID.

4. Alice’s CIDVV platform intercepts each verification call and performs the following actions:

   a. For the call with `+100` prefix: Rejects the call with SIP **486 Busy Here**.

   b. For the call with `+101` prefix: Rejects the call with SIP **603 Call Rejected**.

5. Bob’s CIDVV platform evaluates the responses to both verification calls.

6. If both verification calls receive the expected responses (486 for `+100` and 603 for `+101`) within the Validity Window, Bob’s CIDVV platform:

   a. Considers the vouching successful.

   b. Allows the original call from Alice to proceed and ring through to Bob.

7. If either verification call fails to receive the correct response, times out, or does not complete within the Validity Window, Bob’s CIDVV platform:

   a. Considers the vouching failed.

b. May take any of the following actions on the original call (implementation specific):

   - Reject the call (e.g., with SIP 603 Call Rejected).
   - Route the call to Bob’s voicemail.
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
| +100   | Vouch Phase 1      | Bob → Alice   | Vouching verification (Phase 1)              | 486 Busy Here                |
| +101   | Vouch Phase 2      | Bob → Alice   | Vouching verification (Phase 2)              | 603 Call Rejected            |
| +101   | Wake Call          | Alice → Bob   | Initiate vetting and trigger token generation| 603 Call Rejected            |
| +101   | Recognize Call     | Alice → Bob   | Prove knowledge of shared secret             | 486 Busy Here                |
| +101   | Auth Call          | Bob → Alice   | Prove control of destination number          | 486 Busy Here                |

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

A call using the "100" prefix is the **Phase 1** verification. It is considered successful only if it results in an immediate rejection consistent with a "Busy"-class response (e.g., SIP 486 Busy Here).

### Phase 2 Verification ("101" Prefix)

A call using the "101" prefix is the **Phase 2** verification. For standard vouching, it is expected to result in a "Rejection"-class response (e.g., SIP 603 Call Rejected or SIP 403 Forbidden).

### Combined Phase Behavior (Required for Success)

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

A default timeout of 3–6 seconds is reasonable for domestic calls.
For international destinations, longer timeouts (typically 8–20 seconds)
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
- calling = 12125550100, called = 19495550199, secret = "hamburger"
- Concatenated: "12125550100|19495550199|hamburger"
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

Vetting a remote number requires two separate calls (distinct SIP
dialogs) using a pre-agreed shared key. The process confirms that
the **called party (Bob)** controls the target telephone number and
possesses the correct shared secret. In the examples below, Alice is
the verifier who initiates the two calls to Bob's number in order to
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
SIP response 404 (Not Found).

Alice performs the same SHA-256 calculation and places a second vetting call to Bob. This second call uses a Caller-ID beginning with the Vetting Token Check prefix of "101" followed by the computed numeric code.

When Bob's CIDVV platform receives the Vetting Token Check call, it removes the "101" prefix and compares the remaining numeric code to the recently cached value.

If the numeric code matches, Bob's CIDVV platform MUST reject the call with SIP response 486 (Busy Here). Alice's platform treats this response as a successful vet.

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

In the diagram, "+100" represents a verification call whose
Calling Party Number is "100" (or "+100") plus Bob's Caller-ID

### Successful Vouch Step-by-step Description

The diagram above shows the high-level message flow. The following
numbered steps provide the detailed behavior, including Caller-ID
manipulation performed by CIDVV platforms and CIDVV-aware elements.

1. The originating user (Alice, Caller-ID `+12125550100`) initiates a
   call to Bob (`+19495550199`).

2. The call is routed from Alice’s User Agent to her SBC, which forwards
   it to the originating CIDVV platform (CIDVV_A).

3. **CIDVV_A**:
   - Caches the call attempt using the tuple
     `(Calling: +12125550100, Called: +19495550199)` for the Validity Window.
   - Rejects the call with SIP **404 Not Found**.

4. Alice’s SBC receives the 404 and advances the original call toward
   the PSTN using Alice’s original Caller-ID.

5. The call reaches Bob’s SBC via the PSTN, which forwards it to the
   terminating CIDVV platform (CIDVV_B).

6. **CIDVV_B** initiates a Phase 1 verification call toward Alice’s
   number (`+12125550100`) using the Caller-ID `+10019495550199`.

7. The Phase 1 verification call arrives at Alice’s SBC via the PSTN.

8. **Alice’s SBC**:
   - Detects the leading `+100` prefix on the Caller-ID.
   - Routes the call to CIDVV_A.

9. **CIDVV_A**:
   - Receives the call with `To: +12125550100`, `From: +10019495550199`.
   - Looks up the cached tuple for the Validity Window.
   - Finds a matching entry and recognizes this as a successful **Phase 1 Vouch**.
   - Returns SIP **486 Busy Here**.

10. **CIDVV_B** receives the 486 response, recognizes it as a successful
    Phase 1 verification, and initiates a Phase 2 verification call
    toward Alice using the Caller-ID `+10119495550199`.

11. The Phase 2 verification call arrives at Alice’s SBC via the PSTN.

12. **Alice’s SBC**:
    - Detects the leading `+101` prefix.
    - Routes the call to CIDVV_A.

13. **CIDVV_A** recognizes the `+101` prefix and rejects the call with
    SIP **603 Decline**.

14. **CIDVV_B** receives the 603 response, recognizes it as a successful
    **Phase 2 Vouch**, and considers the combined Phase 1 + Phase 2
    result a **Successful Vouch**.

15. CIDVV_B returns a SIP **302 Moved Temporarily** (with Bob’s Contact)
    to Bob’s SBC.

16. Bob’s SBC receives the 302 and advances the original call to Bob’s
    User Agent.

17. Bob’s telephone rings.

This mechanism allows the originating CIDVV platform to confirm that
the Asserted Caller-ID is valid without completing the initial call.

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
{: title="First vetting call with 101 - creates Recognize Token and responds with 603"}

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
{: title="Second Vetting call with 101 - acknowledge Recognize with 486 Busy Here"}

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
{: title="Second Vetting call with 101 - acknowledge Auth with 486 Busy Here"}

### Successful Caller-ID Vetting Flow

Vetting a remote number requires three separate calls (distinct SIP dialogs) using a pre-agreed shared secret. The process confirms that the called party (Bob) controls the target telephone number and possesses the correct shared secret.

1. Alice and Bob agree on a shared secret (e.g., `hamburger`) and Alice's vetting Caller-ID (`+12125550100`).

2. Both parties enter the shared secret, Alice's vetting Caller-ID, and an optional Validity Window into their respective CIDVV platforms.

3. Alice's CIDVV platform (CIDVV_A) initiates the first vetting call ("Wake Call") with Caller-ID `+10112125550100` toward Bob's number (`+19495550199`). The call traverses the PSTN.

4. Bob's SBC recognizes the leading `+101` prefix on the incoming Caller-ID and forwards the call to Bob's CIDVV platform (CIDVV_B).

5. **CIDVV_B**:
   - Strips the leading `+101`, recovering Alice's vetting Caller-ID.
   - Recognizes this as a pre-agreed vetting Caller-ID.
   - Computes the "Recognize Token" using the algorithm defined in Section [Hash Function for Vetting].
   - Caches this token for the Validity Window.
   - Rejects the call with SIP **603 Decline**.

6. CIDVV_A receives the 603 and performs the identical calculation to derive the Recognize Token.

7. CIDVV_A immediately places the second vetting call ("Recognize Call") to `+19495550199` using Caller-ID `+101` followed by the Recognize Token (example: `+10112953388433`).

8. Bob's SBC recognizes the `+101` prefix and forwards the call to CIDVV_B.

9. **CIDVV_B**:
   - Strips the leading `+101`.
   - Verifies that the remaining digits match the recently cached Recognize Token.
   - Responds with SIP **486 Busy Here** to signal a successful Recognize step.

10. CIDVV_A receives the 486 Busy Here and computes the "Auth Token" using a slightly different derivation (shared secret + called number + calling number).

11. CIDVV_B computes the same Auth Token and initiates the third call ("Auth Call") to Alice (`+12125550100`) using Caller-ID `+101` followed by the Auth Token (example: `+10114728195634`).

12. **CIDVV_A**:
    - Strips the leading `+101`.
    - Verifies that the received Auth Token matches the value it computed.
    - Responds with SIP **486 Busy Here**.
    - Considers the vetting procedure successful and reports success to Alice.

13. **CIDVV_B** receives the 486 Busy Here and also considers the vetting procedure successful.

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
call timeouts should be adaptive: 3–6 seconds is appropriate for domestic
calls, while 8–20 seconds (or more) may be needed for international
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

# Security Considerations

CIDVV verification is probabilistic and based on reachability.
It does not provide cryptographic identity guarantees and is
intended to complement, not replace, mechanisms such as
STIR/SHAKEN.

Its security properties derive from the inability of an attacker
to receive calls at the Asserted Caller-ID (the number being
vouched).

CIDVV does not provide per-call correlation and instead validates
reachability within the Validity Window. This may result in multiple
calls being validated by a single successful vouch.

The use of distinct response patterns across multiple verification
calls (e.g., "100" -> 486 and "101" -> 603) increases resistance to
false-positive validation arising from common network behaviors.

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

## Replay Attacks

CIDVV uses short-lived state (typically on the order of seconds) to
correlate signaling exchanges. This limits the effectiveness of
replay attacks.

Implementations MUST:
- Expire cached state quickly (e.g., within ~10 seconds)
- Reject verification attempts that do not match recent state

Replay within the Validity Window remains theoretically possible but requires
precise timing and routing alignment (see Section <xref target="hash-function"/> for vetting tokens).

## Spoofing Resistance

CIDVV prevents spoofing by requiring the party asserting a Caller-ID
to successfully receive and respond to a return call routed via the
PSTN. An attacker (Mallory) who does not control the corresponding
number cannot receive the verification call and therefore cannot
complete the vouching process.

## Denial of Service

CIDVV introduces additional signaling traffic, which may be abused
for denial-of-service (DoS) purposes.

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
