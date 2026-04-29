---
title: CallerID Vouching and Vetting (CIDVV)
abbrev: CIDVV
category: info
docname: draft-anderson-askew-cidvv-latest
submissiontype: IETF
date: 2026-04
consensus: false
v: 3
keyword:
  - callerid
  - vouching
  - vetting
  - LERG
  - PSTN
  - spoofing

venue:
  github: "jollyrogertelephone/draft-cidvv"

author:
  -
    ins: R. Anderson
    fullname: Roger Anderson
    organization: Jolly Roger Telephone Company
    email: roger@jollyrogertelephone.com
    country: US
  -
    ins: P. Askew
    fullname: Phillip Askew
    organization: ""
    email: ""
    country: US

normative:
informative:
...

--- abstract
This document describes CIDVV (CallerID Vouching and Vetting), a lightweight protocol that uses the existing PSTN and LERG to provide strong vouching and vetting of Caller-ID numbers.

--- middle
# Introduction

Virtually every telephone user worldwide receives spoofed, spam, and scam calls. This document defines CIDVV to restore trust in Caller-ID using only existing PSTN routing and the LERG.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Vouching Mechanism

TODO — detailed 10-prefix flow

# Vetting Mechanism

TODO — detailed 11-prefix flow

# Security Considerations

TODO

# IANA Considerations

This document has no IANA actions.

--- back
# Acknowledgments
{:numbered="false"}

TODO

# Authors' Addresses

Roger Anderson  
Jolly Roger Telephone Company  
Email: roger@jollyrogertelephone.com  

Phillip Askew  
Email: TBD
