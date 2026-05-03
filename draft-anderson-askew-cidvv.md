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
keyword: telephony, callerid, spoofing, PSTN

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
---

<!--
Notes:
Headings (starting with #) must be separated by a blank line. Can use this regex to find: ^#{1,6} .+\n[^\n]
-->

--- abstract

Caller-ID spoofing remains a significant problem in telephony,
particularly across inter-domain and international call paths where
identity frameworks may not be consistently applied.

This document defines Caller-ID Vouching and Vetting (CIDVV), a
lightweight verification mechanism that uses short-lived signaling
exchanges encoded within the Calling Party Number to confirm that a
calling party can receive calls at the asserted number.

CIDVV is designed to operate across heterogeneous SIP and SS7/TDM
networks without requiring new protocol extensions or persistent
identity infrastructure. It relies on existing call routing behavior
and the exchange of failure responses rather than successful call
completion.

The mechanism improves resistance to Caller-ID spoofing by requiring
demonstrable control of the asserted number, while remaining
incrementally deployable and tolerant of intermediate network
modification.

--- middle

# Introduction

Caller-ID spoofing is widely used in fraudulent and nuisance calling,
particularly in environments where calls traverse multiple
administrative domains and heterogeneous network technologies.
Although mechanisms such as STIR/SHAKEN provide cryptographic
attestation of caller identity, their effectiveness is limited by
partial deployment and challenges in international interconnection.

This document defines Caller-ID Vouching and Vetting (CIDVV), a
mechanism that verifies caller identity through network reachability
rather than asserted identity. CIDVV requires that a party claiming a
Caller-ID be able to receive a return call at that number within a
short time window.

CIDVV operates by encoding signaling information within the Calling
Party Number and leveraging existing call routing behavior to perform
a challenge-response exchange. The protocol does not require new SIP
headers, protocol extensions, or changes to SS7 signaling, and is
designed to function across mixed SIP and TDM networks.

CIDVV is incrementally deployable and does not require universal
adoption to provide benefit. It tolerates modification of signaling
by intermediate networks and relies only on the ability of signaling
calls and failure responses to traverse the network path.

CIDVV does not provide absolute identity assurance but significantly
raises the cost of Caller-ID spoofing by requiring demonstrable
control of the asserted number.

CIDVV leverages two key elements of the existing telephone ecosystem:

* Existing routing databases and numbering plans, which provide
  authoritative routing ownership for telephone numbers.
* Digit sequences chosen to minimize conflict with valid numbering plans (e.g., "100" and "101")

The mechanism operates entirely within normal PSTN routing behavior and requires no media exchange.

# Terminology

* **Alice**: The calling party (originator of the call being vetted).
* **Bob**: The called party (owner of the number being vetted).
* **Mallory**: An attacker attempting to spoof a Caller-ID.
* **OSP**: Originating Service Provider.
* **TSP**: Terminating Service Provider.
* **CIDVV Platform**: A system that implements the vouching and vetting procedures defined in this document.
* **CIDVV-aware Network Element**: An SBC or intermediary that recognizes CIDVV signaling prefixes and interprets associated responses, but does not implement the full CIDVV platform logic.
* **Vouch**: The act of a CIDVV platform asserting that it has verified control of a telephone number through the two-call challenge-response mechanism described in this document. A successful vouch proves the calling party legitimately controls the asserted Caller-ID.
* **Vet** (or **Vetting**): The process by which a CIDVV platform confirms legitimate ownership of a telephone number via the two-call challenge-response sequence. Vetting may be performed by the number owner directly or on behalf of third parties such as Caller-ID branding services, Google Business Profiles, trade organizations, or enterprise trust programs.
* **Vouching Call**: One of the two short calls used in the CIDVV protocol (typically rejected with 404 or 486).
* **Successful Vouch**: A verification result indicating that a matching cache entry was found.
* **Unsuccessful Vouch**: A verification result indicating that no matching cache entry was found.
* **Verification Not Performed**: A condition where verification could not be completed due to system or network conditions.

## Motivation and Advantages

The CIDVV vouching and vetting mechanism is designed to operate with minimal new infrastructure while providing strong protection against Caller-ID spoofing.  Its primary advantages are:

* **Leverages existing PSTN infrastructure**: The mechanism uses existing numbering plans and routing databases to direct calls without requiring additional infrastructure.

* **Strong anti-spoofing protection**: A successful vouch proves that the asserted Caller-ID is controlled by the legitimate owner, because only the real owner can generate the correct challenge-response sequence. Spoofed calls are typically rejected early, often resulting in failure responses such as 404 Not Found.

* **Visibility into spoofing activity**: Telephone number owners gain direct insight into how often (and from where) their numbers are being spoofed worldwide through logged vetting attempts.

* **Low signaling overhead**: The two short vetting calls replace what would otherwise be a completed fraudulent call, resulting in lower overall network load.

* **Full TDM/SS7 compatibility**: The mechanism works natively across legacy SS7 and ISDN networks.  SIP is not required.

* **International applicability**: The solution functions across international boundaries without relying on country-specific frameworks.

* **Independent of STIR/SHAKEN**: It provides an effective alternative (or complement) in environments where STIR/SHAKEN is unavailable, not deployed, or insufficient.

* **Enterprise and service-provider flexibility**:
  * Enterprises can deploy their own CIDVV platform using open-source tools such as Kamailio or Asterisk.
  * Service providers or third-party vendors (e.g., Variphy, TransUnion, iconectiv, Somos, TNS, First Orion, or others) can operate cloud-based vouching and vetting services.
  * Customers can easily switch between providers, fostering real competition and driving down costs.

This design lowers the barrier to entry and encourages broad adoption while avoiding the single points of failure and high coordination costs associated with centralized solutions. These properties make the vouching mechanism particularly suitable for service providers, enterprises, and end users who need robust Caller-ID validation today, using only existing telephone infrastructure.

# Design Principles

CIDVV is designed to operate under the assumption that intermediate
networks may normalize, truncate, or otherwise modify signaling
information. The protocol therefore encodes all required signaling in
a numeric Calling Party Number that can survive traversal of mixed
SIP and SS7/TDM networks.

CIDVV relies only on the ability of signaling indicators and failure
responses to traverse the network path; it does not require specific
response codes to be preserved.

# Protocol Overview

CIDVV uses special Caller-ID prefixes to signal protocol operations:

* "100" prefix — Vouching or Vetting
* "101" prefix — Vetting Token Check

CIDVV exchanges occur using short signaling dialogs and do not require media establishment.

CIDVV signaling is encoded entirely within numeric Calling Party
Number values to maximize survivability across heterogeneous SIP and
SS7/TDM networks.

## Response Semantics

CIDVV uses SIP response codes as local signaling indicators between
participating systems.

In typical deployments:

* 486 (Busy Here) indicates a successful vouch or vet.
* 404 (Not Found) indicates an unsuccessful vouch or vet.
* 603 (Decline) indicates that CIDVV is not implemented or that
  verification could not be performed.

However, intermediate SIP and SS7/TDM networks may translate,
modify, or replace response codes. As a result, CIDVV implementations
MUST NOT rely on specific numeric response codes being preserved
end-to-end.

Instead, implementations MUST interpret responses based on observed
behavior and expected signaling patterns (e.g., immediate rejection,
timeout, or call progression).

# Protocol Operation

## Vouching Procedure

Alice's CIDVV platform receives an attempted call from Alice to Bob. It MUST cache the calling number and called number for a short interval, normally about 10 seconds. It then rejects the attempt with SIP response 486 (Busy Here).

Alice's SBC then sends the call normally through the PSTN toward Bob.

When Bob's system receives the call, Bob's system initiates a verification call toward Alice. The verification call uses a Caller-ID formed by prefixing Bob's number with the digits "100".

When Alice's CIDVV platform receives a call with a Caller-ID beginning with "100", it MUST remove the "100" prefix, compare the resulting number pair against the recent cache, and determine whether the original call attempt exists.

If a matching cache entry exists, Alice's CIDVV platform MUST reject
the verification call with SIP response 486 (Busy Here). This
response is interpreted by CIDVV-aware network elements (e.g., SBCs
or intermediaries) as an indication of a successful vouch.

If no matching cache entry exists, Alice's CIDVV platform MUST reject
the verification call with SIP response 404 (Not Found). This
response is interpreted by CIDVV-aware network elements (e.g., SBCs
or intermediaries) as an indication of an unsuccessful vouch.

## Correlation Model

CIDVV vouching correlates calls using the asserted calling number,
the called number, and a short time window. It does not attempt to
identify individual call legs across the PSTN.

If multiple calls with the same asserted calling number and called
number occur within the cache interval, implementations MAY treat
them as a single aggregate vouching state or MAY maintain a count of
pending attempts.

A successful vouch indicates that at least one matching call attempt
occurred during the validity window, rather than proving a
one-to-one correspondence between specific call legs.

## Vetting Procedure

Before vetting begins, Alice and Bob agree on a shared secret, Alice's vetting Caller-ID, and a validity time window.

Alice places a vetting call to Bob using a Caller-ID beginning with the digits "100".

When Bob's CIDVV platform receives the first vetting call, it removes the "100" prefix and verifies that the resulting Caller-ID is expected for the current vetting attempt. Bob's platform then computes a SHA-256 value over the called number followed by the shared secret. Bob's platform converts that value to decimal form, extracts a fixed-length numeric code, stores the code briefly, and rejects the call with SIP response 404 (Not Found).

Alice performs the same SHA-256 calculation and places a second vetting call to Bob. This second call uses a Caller-ID beginning with the Vetting Token Check prefix of "101" followed by the computed numeric code.

When Bob's CIDVV platform receives the Vetting Token Check call, it removes the "101" prefix and compares the remaining numeric code to the recently cached value.

If the numeric code matches, Bob's CIDVV platform MUST reject the call with SIP response 486 (Busy Here). Alice's platform treats this response as a successful vet.

Any other response, timeout, code mismatch, expired cache entry, or unexpected Caller-ID MUST be treated as an unsuccessful vet.

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
     |           |           |- INVITE ->|           |           |
     |           |           |  Step 4   |           |           |
     |           |           |           |- INVITE ->|           |
     |           |           |           |  Step 5   |           |
     |           |           |           |           |           |
     |           |           |           |<- INVITE -|           |
     |           |           |           |  Step 6   |           |
     |           |           |<- INVITE -|           |           |
     |           |           |  Step 7   |           |           |
     |           |<- INVITE -|           |           |           |
     |           |  Step 8   |           |           |           |
     |           |           |           |           |           |
     |           |-- Busy -->|           |           |           |
     |           |  Step 9   |           |           |           |
     |           |           |-- Busy -->|           |           |
     |           |           |  Step 10  |           |           |
     |           |           |           |-- Busy -->|           |
     |           |           |           |  Step 11  |           |
     |           |           |           |           |- INVITE ->|
     |           |           |           |           |  Step 12  |
     |           |           |           |           |           |
~~~~
{: #fig-successful-vouch title="Example Successful Vouch"}

### Successful Vouch Step-by-step description

The diagram above shows the high-level message flow. The following numbered steps provide the detailed behavior, including Caller-ID manipulation performed by the CIDVV platforms.

1. The originating user (Alice, Caller-ID `+12125550100`) initiates a call to the destination user (Bob, dialed number `+19495550199`).

2. The call is routed from Alice's User Agent to her SBC, which forwards it to the originating CIDVV platform (CIDVV_A).

3. **CIDVV_A**:
   - Caches the call attempt `(From: +12125550100, To: +19495550199)` for a short period (≈10 seconds).
   - Rejects the call with **486 Busy Here**.

4. Alice's SBC receives the 486 and advances the original call toward the PSTN using the original Caller-ID.

5. The call reaches Bob's SBC via the PSTN.

6. **Bob's SBC**:
   - Generates a Calling Party Number of `100` plus the last 12 digits of the dialed number, resulting in `10019495550199`.
   - Forwards the call back toward Alice's number (`+12125550100`) with the modified Caller-ID `+10019495550199`.

7. The return call arrives at Alice's SBC via the PSTN.

8. **Alice's SBC**:
   - Detects the leading `100` prefix on the Caller-ID.
   - Routes the call to CIDVV_A for vouch verification.

9. **CIDVV_A**:
   - Receives the call with `To: +12125550100` and `From: +10019495550199`.
   - Strips the `100` prefix from the From number, yielding `+19495550199`.
   - Swaps the From and To values.
   - Looks up the resulting pair `(+12125550100, +19495550199)` in its short-term cache.
   - Finds a matching entry from step 3.
   - Considers this a successful vouch and returns **486 Busy Here**.

10. Bob's SBC receives the 486 via the PSTN, recognizes it as a successful vouch response, and advances the original call to Bob's User Agent.

11. Bob's telephone rings.

This two-call mechanism (first vetting call + return vouch call) allows the originating CIDVV platform to confirm that the asserted Caller-ID is valid without completing the initial call.

## Unsuccessful Vouch Call Flow

The following diagram shows an unsuccessful vouch attempt by an impersonator (Mallory) who spoofs Alice's Caller-ID.

~~~~
  Mallory     CIDVV_A      SBC_A        PSTN       SBC_B    Voicemail_B
     |           |           |           |           |           |
     |------------- INVITE ------------->|           |           |
     |  Step 1   |           |           |           |           |
     |           |           |           |- INVITE ->|           |
     |           |           |           |  Step 2   |           |
     |           |           |           |           |           |
     |           |           |           |<- INVITE -|           |
     |           |           |           |  Step 3   |           |
     |           |           |<- INVITE -|           |           |
     |           |           |  Step 4   |           |           |
     |           |<- INVITE -|           |           |           |
     |           |  Step 5   |           |           |           |
     |           |           |           |           |           |
     |           |-NotFound->|           |           |           |
     |           |  Step 6   |           |           |           |
     |           |           |-NotFound->|           |           |
     |           |           |  Step 7   |           |           |
     |           |           |           |-NotFound->|           |
     |           |           |           |  Step 8   |           |
     |           |           |           |           |--- VM --->|
     |           |           |           |           |  Step 9   |
     |           |           |           |           |           |
~~~~
{: #fig-unsuccessful-vouch title="Example Unsuccessful Vouch"}

### Unsuccessful Vouch Step-by-step description

1. Mallory spoofs Alice’s Caller-ID (`+12125550100`) and initiates a call to Bob (`+19495550199`).

2. The call arrives at Bob’s SBC via the PSTN.

3. **Bob’s SBC**:
   - Generates a Calling Party Number of `100` plus the last 12 digits of the dialed number, resulting in `10019495550199`.
   - Forwards the call toward Alice’s number (`+12125550100`) with the modified Caller-ID `+10019495550199`.

4. The call arrives at Alice’s SBC via the PSTN.

5. **Alice’s SBC**:
   - Detects the `100` prefix and recognizes this as a vouch call.
   - Routes the call to CIDVV_A for verification.

6. **CIDVV_A**:
   - Processes `To: +12125550100` and `From: +10019495550199`.
   - Strips the `100` prefix, yielding `+19495550199`.
   - Swaps From and To values.
   - Looks up the pair `(+12125550100, +19495550199)` in its short-term cache.
   - Finds no matching entry (because Alice never placed the original call).
   - Rejects the call with **404 Not Found**.

7. The 404 propagates back through the PSTN to Bob’s SBC.

8. **Bob’s SBC** recognizes the 404 as a unsuccessful vouch and either rejects the call or forwards it to Bob’s voicemail. Bob remains unaware of the impersonation attempt.

This mechanism ensures that only calls that originated from a legitimate CIDVV platform (i.e., those that previously cached the attempt) will pass vouching. Spoofed or unsolicited calls are rejected early.

## Vetting a Caller-ID Number

Vetting a number requires **two independent calls** (separate SIP dialogs). The first call checks whether the number is known; the second call performs the confirmation.

### First Vetting Call

~~~~
   CIDVV_A        SBC_A          PSTN         SBC_B        CIDVV_B
      |             |             |             |             |
      |-- INVITE -->|             |             |             |      Step 1
      |   Step 1    |             |             |             |
      |             |-- INVITE -->|             |             |
      |             |   Step 2    |             |             |
      |             |             |-- INVITE -->|             |
      |             |             |   Step 3    |             |
      |             |             |             |-- INVITE -->|
      |             |             |             |   Step 4    |
      |             |             |             |             |
      |             |             |             |<-Not Found--|
      |             |             |             |   Step 5    |
      |             |             |<-Not Found--|             |
      |             |             |   Step 6    |             |
      |             |<-Not Found--|             |             |
      |             |   Step 7    |             |             |
      |<-Not Found--|             |             |             |
      |   Step 8    |             |             |             |
      |             |             |             |             |
~~~~
{: title="First vetting call with 100 - creates cache entry or receives 404"}

### Second Vetting Call

~~~~
   CIDVV_A        SBC_A          PSTN         SBC_B        CIDVV_B
      |             |             |             |             |
      |-- INVITE -->|             |             |             |
      |   Step 1    |             |             |             |
      |             |-- INVITE -->|             |             |
      |             |   Step 2    |             |             |
      |             |             |-- INVITE -->|             |
      |             |             |   Step 3    |             |
      |             |             |             |-- INVITE -->|
      |             |             |             |   Step 4    |
      |             |             |             |             |
      |             |             |             |<-Busy Here--|
      |             |             |             |   Step 5    |
      |             |             |<-Busy Here--|             |
      |             |             |   Step 6    |             |
      |             |<-Busy Here--|             |             |
      |             |   Step 7    |             |             |
      |<-Busy Here--|             |             |             |
      |   Step 8    |             |             |             |
      |             |             |             |             |
~~~~
{: title="Vetting token check call with 101 - confirms vouch with 486 Busy Here"}

### Successful Caller-ID Vetting Flow

Vetting a remote number requires two separate calls (distinct SIP dialogs) using a pre-agreed shared key. The process confirms that the called party controls the target telephone number and possesses the correct shared secret.

1. Alice and Bob agree on a shared secret (e.g. `hamburger`) and Alice’s vetting Caller-ID (`+12125550100`).

2. Both parties enter the shared secret, Alice’s vetting Caller-ID, and an optional validity window (e.g. one week) into their respective CIDVV platforms.

3. Alice’s CIDVV platform (CIDVV_A) initiates the first vetting call with Caller-ID `+10012125550100` toward Bob’s number (`+19495550199`). The call traverses the PSTN.

4. Bob’s SBC recognizes the leading `100` prefix on the incoming Caller-ID and forwards the call to Bob’s CIDVV platform (CIDVV_B).

5. **CIDVV_B**:
   - Strips the leading `100`, recovering Alice’s Caller-ID `+12125550100`.
   - Recognizes this as a pre-agreed Vetting Caller-ID
   - Computes the SHA-256 digest of the concatenated string
     `+12125550100+19495550199hamburger`.
   - Takes the first 8 hexadecimal characters (`b0092191`), converts to decimal (`2953388433`), pads to 10 digits, and prepends `1`, yielding `12953388433`.
   - Caches this value for a short period (≈10 seconds).
   - Rejects the call with **404 Not Found**.

6. CIDVV_A receives the 404 and performs the identical hash calculation to derive `12953388433`.

7. CIDVV_A immediately places a second vetting call to `+19495550199` using the Vetting Token Check Caller-ID `+10112953388433`.

8. Bob’s SBC recognizes the `101` prefix on the Caller-ID and forwards the call to CIDVV_B.

9. **CIDVV_B**:
   - Recognizes the leading `101` as a Vetting Token Check call
   - Strips the leading `101`.
   - Observes that the remaining Caller-ID (`12953388433`) matches a recently cached vetting token.
   - Responds with **486 Busy Here** to signal a successful vet.

10. CIDVV_A receives the 486 Busy Here and reports a successful vet to Alice.

#### Vetting Failure Cases

A vetting attempt may fail for the following reasons:

* Bob does not have a participating CIDVV platform — the first call will not return 404, or the second call will not return 486.
* The shared secret, Alice’s vetting Caller-ID, or time window does not match — the two calls will not produce the expected 404 + 486 sequence.
* Network or policy restrictions prevent one or both calls from reaching the remote CIDVV platform.

This two-call challenge-response mechanism provides strong confirmation that the remote number is both reachable via the PSTN and controlled by an entity that knows the shared secret.

# Deployment Considerations

## Behavior of Non-CIDVV Systems

Systems that do not implement CIDVV are not expected to recognize
CIDVV signaling prefixes. Such systems will typically process CIDVV
calls as ordinary calls and may return a wide range of responses.

CIDVV implementations MUST treat any response that does not match the
expected protocol behavior as indicating a non-participating system.

## Handling of CIDVV Signaling Calls

Networks that recognize CIDVV SHOULD NOT present calls with Calling
Party Numbers beginning with "100" or "101" to end users.

Such calls SHOULD be intercepted by network elements or CIDVV
platforms and SHOULD result in a non-success response (e.g., 4xx,
5xx, or 6xx class response codes). Implementations commonly use
responses such as 486 (Busy Here) or 603 (Decline).

Call analytics, labeling, and fraud detection systems SHOULD
recognize CIDVV signaling prefixes ("100" and "101") and treat such
calls as protocol signaling rather than ordinary subscriber calls.

CIDVV signaling calls are not intended to complete. Implementations
SHOULD minimize call duration and signaling load and SHOULD avoid any
media establishment.

## Response Variability

CIDVV implementations MUST assume that response codes may be altered,
mapped, or replaced by intermediate SIP or SS7/TDM networks. As a
result, implementations MUST NOT rely on any specific response code
being preserved end-to-end.

Instead, implementations SHOULD interpret responses based on expected
classes of behavior (e.g., success vs. failure) rather than exact
numeric values.

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

SIP intermediaries and SBCs that support CIDVV SHOULD preserve the
Calling Party Number digits used for CIDVV signaling, including the
leading "100" or "101" prefix, across trusted interfaces unless local
policy explicitly rejects the call.

CIDVV does not rely on Type-of-Number (TON) preservation and assumes
that intermediate networks may normalize or reinterpret numbering
format.

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

CIDVV relies on short-lived signaling exchanges and does not require
persistent identity infrastructure. Its security properties are
derived from control of telephone number routing and the ability to
complete a two-call challenge-response sequence.

CIDVV does not provide per-call correlation and instead validates
reachability within a short time window. This may result in multiple
calls being validated by a single successful vouch.

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

Replay within the validity window remains theoretically possible but
requires precise timing and routing alignment.

## Spoofing Resistance

CIDVV prevents spoofing by requiring the party asserting a Caller-ID
to successfully receive and respond to a return call routed via the
PSTN. An attacker that cannot receive calls to the claimed number
cannot complete the vouching process.

However, CIDVV does not prevent attacks where:
- The attacker controls the terminating endpoint for the number
- The attacker has compromised the service provider infrastructure

## Denial of Service

CIDVV introduces additional signaling traffic, which may be abused
for denial-of-service (DoS) purposes.

Implementations MUST:
- Rate-limit CIDVV signaling requests
- Detect and suppress repeated unsuccessful attempts
- Bound resource usage for temporary state

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

CIDVV does not rely on specific SIP response codes being preserved
end-to-end. Intermediate networks may translate or modify response
codes.

Implementations MUST interpret responses based on expected behavior
(success vs. failure) rather than exact numeric values.

## Data Privacy

CIDVV exchanges inherently expose calling and called numbers within
signaling messages.

Implementations SHOULD:
- Avoid storing telephone numbers in plaintext where possible
- Use derived values (e.g., cryptographic hashes) for temporary state
- Limit retention of any identifying data

Temporary state MUST be short-lived and automatically expired.

## Hash-Based Token Security (Vetting)

Vetting operations use shared secrets and derived tokens.

Implementations MUST:
- Use cryptographically secure hash functions (e.g., SHA-256)
- Protect shared secrets from disclosure
- Ensure tokens are valid only within a short time window

Implementations SHOULD:
- Include sufficient entropy in derived tokens
- Avoid predictable or reusable values

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
absolute identity assurance. It should be considered a probabilistic
verification mechanism that significantly raises the cost of
spoofing attacks rather than eliminating them entirely.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

The authors thank contributors to telephony security research and PSTN infrastructure development.
