# CIDVV: Caller-ID Vouching and Vetting  
**A Practical, Lightweight Solution to Caller-ID Spoofing**

**Version 1.0**  
**May 2026**  

**Sponsored by Jolly Roger Telephone Company**

**Roger Anderson** – Jolly Roger Telephone Company  
**Steven Berkson** – Jolly Roger Telephone Company  
**Phillip Askew** – Jolly Roger Telephone Company  

## Executive Summary

Caller-ID spoofing continues to erode trust in telephony. Despite years of effort and significant investment in STIR/SHAKEN, spoofed calls remain common — especially across international borders and legacy TDM networks.

**CIDVV (Caller-ID Vouching and Vetting)** offers a simple, immediately deployable complement that that verifies reachability and control of the presented number. It requires **no new protocol extensions**, works on both SIP and SS7/TDM networks, and can be rolled out incrementally.

By using short-lived signaling exchanges encoded in the existing Calling Party Number field, CIDVV raises the cost of spoofing dramatically while creating new opportunities for verified calling services, branded numbers, and trust programs.

## The Problem

- Robocalls and spoofing scams cost billions annually.
- International and TDM calls often lack reliable identity.
- STIR/SHAKEN provides strong attestation where fully deployed, but coverage remains incomplete globally.
- Sophisticated scammers and robocallers often intentionally route calls through TDM/SS7 segments to strip or downgrade STIR/SHAKEN attestation, exploiting legacy infrastructure.
- Legitimate callers (enterprises, banks, healthcare providers) struggle to reach customers because their numbers are frequently blocked or labeled as spam.

## The CIDVV Solution

CIDVV is a lightweight network-native challenge-response mechanism that verifies **reachability and control** of an asserted Caller-ID using the existing telephone network.

In CIDVV, “vouching” means that the called party can confirm that
the owner of the asserted Caller-ID is reachable and has effectively
endorsed the call by responding to a verification signal.

CIDVV does not replace STIR/SHAKEN, but complements it by providing
verification in environments where attestation is unavailable or
unreliable.

[Insert Baseline Vouch Ladder Diagram Here]

[Insert Vetting Flow Diagrams Here]

### Technical Highlights
- Fully compatible with SIP and legacy SS7/TDM
- No new headers or signaling extensions required
- Survives intermediate network normalization and truncation
- Short validity window (on the order of seconds) limits replay risk

## Key Benefits

| Stakeholder       | Benefit |
|-------------------|--------|
| **Carriers**      | Reduced robocalls, lower complaint volume, new revenue from verified calling tiers |
| **Enterprises**   | Reliable delivery of customer-service calls, protected brand reputation |
| **Number Owners** | Visibility into who is spoofing their number and the ability to prove legitimacy |
| **Vendors**       | New service offerings and competitive differentiation |

## Business & Ecosystem Opportunities

### Tier-1 Vendors

Large identity and analytics providers such as TNS, TransUnion (Neustar), First Orion, and Hiya can quickly integrate CIDVV into their existing platforms.

### Opportunities for Tier-2 and Emerging Vendors
CIDVV is especially attractive to secondary and emerging vendors (such as Numeracle, NumHub, and other specialized reputation, KYC, and remediation providers) who want to expand their capabilities or enter the market. Its low deployment barriers allow Tier-2 companies to rapidly enter the market and differentiate through niche verticals or more agile offerings.

### International Opportunities
STIR/SHAKEN has seen very limited adoption outside North America. CIDVV is uniquely positioned for global use because it is TDM/SS7-native and incrementally deployable.

High-potential regions include Europe (Vodafone, Orange, Telefónica), Canada, India, and Latin America. Carriers and vendors in these markets can offer “Verified International Calling” services that are not possible with STIR/SHAKEN alone.

## Call for Sponsors & Partners

Jolly Roger Telephone Company is actively seeking partners to help drive CIDVV adoption:

- **Carriers and Service Providers** interested in piloting CIDVV
- **Tier-1 and Tier-2 vendors** looking to expand their caller identity and anti-fraud offerings
- **Industry organizations** (GSMA, INCOMPAS, etc.) interested in standardization support
- **Enterprise users** and vertical organizations (finance, healthcare, government) that want verified calling

We offer technical collaboration, early access to reference implementations, and co-marketing opportunities.

## Next Steps

1. Review the full specification: [draft-anderson-askew-cidvv](https://jollyrogertelephone.github.io/draft-cidvv/)
2. Contact us to discuss a pilot or collaboration
3. Explore the open-source reference implementation (coming soon)

**Contact**  
Roger Anderson  
roger@jollyrogertelephone.com  
Jolly Roger Telephone Company

---

**About the Sponsor**  
**Jolly Roger Telephone Company** is an independent U.S. carrier focused on innovative, practical solutions to combat telephony fraud while protecting legitimate calling. We developed CIDVV to solve real operational problems we face every day in mixed SIP and TDM environments.

---

**CIDVV provides a practical path to improving trust in telephony, starting today, across both SIP and TDM environments.**
