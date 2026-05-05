---
title: CIDVV - Caller-ID Vouching and Vetting
layout: default
---

# Caller-ID Vouching and Vetting (CIDVV)

**A lightweight, incrementally deployable mechanism to strongly reduce Caller-ID spoofing using existing PSTN and SIP routing behavior.**

CIDVV proves control of an Asserted Caller-ID through short-lived challenge-response signaling calls — no new protocol extensions, no cryptography required, and full compatibility with SIP, SS7, and TDM networks.

---

## Current Specification

**Latest Draft**: May 2026

- **[HTML](draft-anderson-askew-cidvv.html)** ← Recommended reading
- **[TXT](draft-anderson-askew-cidvv.txt)**
- **[PDF](draft-anderson-askew-cidvv.pdf)** (generated via xml2rfc)
- **[Markdown Source](draft-anderson-askew-cidvv-latest.md)** (working copy)
- **[XML](draft-anderson-askew-cidvv.xml)** (for IETF submission)

**Repository**: [github.com/Jolly-Roger-Telephone-Company/cidvv-spec](https://github.com/Jolly-Roger-Telephone-Company/cidvv-spec)

---

## Why CIDVV?

- Works today across mixed SIP/TDM/international paths
- Strong anti-spoofing via network reachability
- Very low signaling overhead
- Independent of (and complementary to) STIR/SHAKEN
- Easy to implement in Asterisk, Kamailio, FreeSWITCH, etc.
- Open, vendor-neutral, and competition-friendly

---

## Quick Links

- [Full Specification (HTML)](draft-anderson-askew-cidvv.html)
- [Call Flow Examples](examples/)
- [Reference Material](reference/)
- [Implementation Repositories](https://github.com/Jolly-Roger-Telephone-Company?q=cidvv&type=all)
- [Contributing](CONTRIBUTING.md)
- [License (Apache 2.0)](LICENSE)

---

## Get Involved

We are actively seeking:
- Early implementers (Asterisk / Kamailio / carrier labs)
- Sponsors and reference deployment partners
- Feedback from telcos, anti-fraud vendors, and IETF participants

**Contact**: Roger Anderson – roger@jollyrogertelephone.com

---

*Maintained by [Jolly Roger Telephone Company](https://jollyrogertelephone.com) as an open community standards effort.*
