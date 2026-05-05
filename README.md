# Caller-ID Vouching and Vetting (CIDVV)

**An incrementally deployable mechanism to reduce Caller-ID spoofing using existing PSTN/SIP routing behavior.**

CIDVV provides a lightweight challenge-response verification method that proves control of an Asserted Caller-ID without requiring new protocol extensions, cryptographic infrastructure, or universal adoption. It works across SIP, SS7, and TDM networks.

---

## Status

- **Current Version**: draft-anderson-askew-cidvv-latest (May 2026)
- **Intended Status**: Informational (IETF)
- **Repository**: https://github.com/JollyRogerTelephone/cidvv-spec
- **Rendered Draft**: [HTML](https://jollyrogertelephone.github.io/cidvv-spec/draft-anderson-askew-cidvv.html) (GitHub Pages – coming soon)
- **Latest PDF**: (will be generated automatically)

This is a **work in progress** Internet-Draft. Feedback and contributions are welcome.

---

## Overview

Caller-ID spoofing continues to plague telephony networks. While STIR/SHAKEN provides strong cryptographic attestation where fully deployed, many networks (especially international and TDM) still lack coverage.

**CIDVV** offers a practical, complementary solution:
- Uses short-lived signaling calls encoded in the Calling Party Number
- Leverages existing call routing and distinct failure responses (e.g., 486 Busy Here vs 404 Not Found)
- Requires no media, no new headers, and minimal infrastructure
- Works with Asterisk, Kamailio, and other SIP platforms
- Provides strong evidence of number control while remaining tolerant of intermediate network modifications

---

## Repository Contents

- `draft-anderson-askew-cidvv.md` – The main Internet-Draft (markdown format)
- `examples/` – Call flow diagrams and test cases
- `reference/` – Related RFCs and background material
- `implementation/` – (planned) Reference scripts and modules

---

## How to Contribute

We welcome contributions from the telephony, SIP, and anti-spoofing communities.

1. Fork this repository
2. Create a new branch for your changes
3. Submit a Pull Request with a clear description
4. For major changes, please open an Issue first to discuss

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

**Note**: All contributions are under the [Apache 2.0 License](LICENSE) and subject to the IETF Note Well.

---

## Implementation Repositories (Planned)

- `cidvv-asterisk` – Asterisk modules and dialplan examples
- `cidvv-kamailio` – Kamailio modules and Lua scripts
- `cidvv-reference` – Lightweight reference server (Node.js / Go)

---

## Related Resources

- Domain: [cidvv.org](https://cidvv.org) (coming soon)
- Jolly Roger Telephone Company – https://jollyrogertelephone.com
- Authors:
  - Roger Anderson ([roger@jollyrogertelephone.com](mailto:roger@jollyrogertelephone.com))
  - Phillip Askew

---

## License

Copyright © 2025-2026 Roger Anderson, Phillip Askew, and contributors.

This work is licensed under [Apache License 2.0](LICENSE).

---

**We are actively seeking sponsors, implementers, and early testers.**  
If your organization is interested in hosting a reference implementation, providing feedback, or supporting this work toward RFC status, please contact roger@jollyrogertelephone.com.

---

*This repository is maintained by Jolly Roger Telephone Company as an open, community-driven standards effort.*
