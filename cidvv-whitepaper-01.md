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

Carriers, enterprises, Tier-1 vendors, and emerging Tier-2 providers can all benefit.

## The Problem

- Robocalls and spoofing scams cost billions annually.
- International and TDM calls often lack reliable identity.
- STIR/SHAKEN provides strong attestation where fully deployed, but coverage remains incomplete globally.
- Many legitimate callers (enterprises, banks, healthcare providers) struggle to reach customers because their numbers are blocked or labeled as spam.

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
- Short 10-second validity window limits replay risk

## Key Benefits

| Stakeholder       | Benefit |
|-------------------|--------|
| **Carriers**      | Reduced robocalls, lower complaint volume, new revenue from verified calling tiers |
| **Enterprises**   | Reliable delivery of customer-service calls, protected brand reputation |
| **Number Owners** | Visibility into who is spoofing their number and the ability to prove legitimacy |
| **Vendors**       | New service offerings and competitive differentiation |

## Deployment & Adoption Path

CIDVV is explicitly designed for **incremental deployment**:
- A single carrier or large enterprise can start benefiting immediately.
- CIDVV-aware elements can be deployed as SBC features or standalone platforms.
- Open-source reference implementations (Kamailio, Asterisk) will be available.
- Works alongside STIR/SHAKEN — carriers can use both.

No “flag day” or universal adoption required.

## Business & Ecosystem Opportunities

### Tier-1 Vendors
Large players such as TNS, TransUnion (Neustar), First Orion, and Hiya can quickly integrate CIDVV into their existing platforms and offer enhanced vouching and vetting as premium services.

### Opportunities for Tier-2 and Emerging Vendors
CIDVV is especially attractive to secondary and emerging vendors (such as Numeracle and other specialized reputation, KYC, and remediation providers) who are looking to catch up or create new revenue streams.  

Because CIDVV has **low deployment barriers** and can be offered as a lightweight cloud service, Tier-2 companies can rapidly enter the market without massive infrastructure investment. They can differentiate by focusing on niche verticals (e.g., healthcare, finance, government) or by offering more flexible, lower-cost, or faster-to-deploy solutions than the dominant Tier-1 players. This levels the playing field and encourages real competition, ultimately driving innovation and lower costs for carriers and enterprises.

### International Opportunities
STIR/SHAKEN has seen very limited adoption outside North America. Most countries still rely on voluntary guidelines, CLI block lists, and operator-specific tools, leaving a significant gap in global caller-ID trust.

CIDVV is uniquely positioned to fill this gap because it is:
- Fully TDM/SS7 native
- Works across borders without requiring new international PKI infrastructure
- Incrementally deployable by individual operators or regional groups

**High-potential regions and partners** include:
- **Europe**: Vodafone Group, Orange, Deutsche Telekom, Telefónica, BT Group, and GSMA Fraud & Security Group (FASG)
- **Canada**: Telus, Bell, Rogers
- **India & Asia-Pacific**: Reliance Jio, Airtel, NTT Docomo, Singtel
- **Latin America**: Telefónica (Movistar), América Móvil

Carriers and vendors in these markets can offer “Verified International Calling” services and branded trust programs that are simply not possible with STIR/SHAKEN alone.

## Next Steps

1. **Technical Evaluation** – Review the full specification (draft-anderson-askew-cidvv).
2. **Pilot Program** – We are actively seeking carrier, enterprise, and vendor partners for initial trials (US and international).
3. **Open Source** – Reference implementations and test tools will be published on GitHub.
4. **Feedback** – We welcome comments from the industry.

**Contact**  
Roger Anderson  
roger@jollyrogertelephone.com  
Jolly Roger Telephone Company

---

**We believe CIDVV can meaningfully reduce spoofing and restore trust in telephony — starting today, both in the US and around the world.**

---
