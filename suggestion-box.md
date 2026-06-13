# CIDVV Suggestion Box & To-Do List

## Open Items (High Priority)

- [ ] if bob has a cidvv platform, can he just send all his calls to it? The cidvv platform should act as a vouchee also.
- [ ] Carriers should CAC cidvv calls
- [ ] Carriers should not play zip tones on cidvv calls
- [ ] Call the vouch and vet processes "phase 1" and "phase 2" each has two phases and both must complete.
- [ ] Figure out how to prevent a message storm associated with retries from 404 and 486. These will be "busy-class" (486/600) and "rejection-class" (403/603)
- [ ] Edge cases
  - Diverted calls - the CIDVV platform should check for a vouch against the diverted number first. If that fails, then do the to number
  - Switched Tollfree - I think vetting can identify switched tollfree. It means the vetting token should not expire in case you want to make two vetting calls (where the second is to the switched tollfree).
  - Can a cidvv platform vet itself? Probably not, because it requires the pstn and there's no way for a vetting platform to know if its call went through the pstn. We may need a way for a cidvv platform to establish a trust relationship with a cloud cidvv platform so the cloud can vet the switched tollfree and communicate that to the cidvv platform. This allows cidvv to vouch for switched tollfree calls. Maybe it doesn't matter for an originator. Maybe the originator just vouches for the DID and the tollfree. It's in nobody's interest to vouch incorrectly. It's important for the vetter to get it right.
  - Number porting. Especially internationally. In the EU it's possible to port across countries. There's a central ported routing table. I don't think this has any effect on CIDVV
  - EU toll free or non-geographic like 0800 and 0900.
- [ ] Bob will need a way to "never vouch" certain numbers. Trading floors want immediate ring. Some callerids will be trusted. Callerid, regex, signers, issuers, attestation levels, cnam. There should be a way to bypass the vouching for specific numbers and immediately return a 302 or 404. Maybe there should be a response code specifically for this.
