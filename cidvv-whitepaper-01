# CIDVV: Caller-ID Vouching and Vetting  
**A Practical, Lightweight Solution to Caller-ID Spoofing**

**Version 1.0**  
**May 2026**  
**Roger Anderson** – Jolly Roger Telephone Company  
**Phillip Askew**

## Executive Summary

Caller-ID spoofing continues to erode trust in telephony. Despite years of effort and significant investment in STIR/SHAKEN, spoofed calls remain common — especially across international borders and legacy TDM networks.

**CIDVV (Caller-ID Vouching and Vetting)** offers a simple, immediately deployable complement that proves a caller actually controls the number they are presenting. It requires **no new protocol extensions**, works on both SIP and SS7/TDM networks, and can be rolled out incrementally.

By using short-lived signaling exchanges encoded in the existing Calling Party Number field, CIDVV raises the cost of spoofing dramatically while creating new opportunities for verified calling services, branded numbers, and trust programs.

Carriers, enterprises, and third-party vendors can deploy CIDVV today and begin offering high-assurance caller identity services that STIR/SHAKEN alone cannot deliver.

## The Problem

- Robocalls and spoofing scams cost billions annually.
- International and TDM calls often lack reliable identity.
- STIR/SHAKEN provides strong attestation where fully deployed, but coverage remains incomplete.
- Many legitimate callers (enterprises, banks, healthcare providers) struggle to reach customers because their numbers are blocked or labeled as spam due to widespread spoofing of those same numbers.

Number owners currently have **no practical way** to prove control of their own number to the broader network in real time.

## The CIDVV Solution

CIDVV is a lightweight challenge-response mechanism that verifies **reachability and control** of an asserted Caller-ID using the existing telephone network.

It works by:
- Encoding simple protocol signals in the Calling Party Number (using “100” and “101” prefixes).
- Performing short verification calls that never ring a phone.
- Using distinct failure responses (Busy vs. Not Found) as the signaling channel.

Only the legitimate owner of a number can consistently pass the CIDVV test.

CIDVV is deliberately **not** a replacement for STIR/SHAKEN — it is a practical companion that fills the gaps today.

## How CIDVV Works

### Core Mechanism (Vouching)
When a call is placed, the originating CIDVV platform deposits a short-lived token. The terminating side performs a quick verification call back using the special prefix. The originating side responds with the expected behavior only if it controls the number.

The entire exchange completes in seconds and adds minimal delay.

[Insert Baseline Vouch Ladder Diagram Here]

### Higher-Assurance Mode
An optional secondary verification call provides even stronger confidence.

### Vetting (Pre-Shared Secret)
For branding programs, Google Business Profiles, or enterprise trust lists, a two-call challenge using a shared secret allows a number owner to proactively prove control.

[Insert Vetting Flow Diagrams Here]

### Technical Highlights
- Fully compatible with SIP and legacy SS7/TDM
- No new headers or signaling extensions required
- Survives intermediate network normalization and truncation
- Uses existing routing databases — no central database needed
- Short 10-second validity window limits replay risk

## Key Benefits

| Stakeholder       | Benefit |
|-------------------|--------|
| **Carriers**      | Reduced robocalls, lower complaint volume, new revenue from verified calling tiers |
| **Enterprises**   | Reliable delivery of customer-service calls, protected brand reputation |
| **Number Owners** | Visibility into who is spoofing their number and the ability to prove legitimacy |
| **Vendors**       | New service offerings (TransUnion, TNS, First Orion, Hiya, Numeracle, etc.) |
| **Consumers**     | Fewer scam calls, higher trust in incoming numbers |

## Deployment & Adoption Path

CIDVV is explicitly designed for **incremental deployment**:
- A single carrier or large enterprise can start benefiting immediately.
- CIDVV-aware elements can be deployed as SBC features or standalone platforms.
- Open-source reference implementations (Kamailio, Asterisk) will be available.
- Works alongside STIR/SHAKEN — carriers can use both.

No “flag day” or universal adoption required.

## Business & Ecosystem Opportunities

- **Verified Calling Services** – Carriers offer premium “Verified” branding.
- **Trust Programs** – Trade associations, banks, and government agencies can run vetting programs.
- **Analytics & Insights** – Number owners see real-time spoofing attempts.
- **Competitive Differentiation** – Early adopters gain a clear advantage in call completion rates.

## Next Steps

1. **Technical Evaluation** – Review the full specification (draft-anderson-askew-cidvv).
2. **Pilot Program** – We are actively seeking 2–3 carrier or large enterprise partners for initial trials.
3. **Open Source** – Reference implementations and test tools will be published on GitHub.
4. **Feedback** – We welcome comments from the industry.

**Contact**  
Roger Anderson  
roger@jollyrogertelephone.com  
Jolly Roger Telephone Company

---

**We believe CIDVV can meaningfully reduce spoofing and restore trust in telephony — starting today.**

---
