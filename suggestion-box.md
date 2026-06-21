# CIDVV Suggestion Box & To-Do List

## Open Items (High Priority)

- [X] if bob has a cidvv platform, can he just send all his calls to it? The cidvv platform should act as a vouchee also.
- [ ] Carriers should CAC cidvv calls - i.e. limit vetting calls to 1 cps or something
- [ ] Carriers should not play zip tones on cidvv calls
- [X] Call the vouch and vet processes "phase 1" and "phase 2" each has two phases and both must complete.
- [X] Figure out how to prevent a message storm associated with retries from 404 and 486. These will be "busy-class" (486/600) and "rejection-class" (403/603)
- [ ] Edge cases
  - [ ] Diverted calls - the CIDVV platform should check for a vouch against the diverted number first. If that fails, then do the to number
  - [ ] Switched Tollfree - I think vetting can identify switched tollfree. It means the vetting token should not expire in case you want to make two vetting calls (where the second is to the switched tollfree).
  - [ ] Can a cidvv platform vet itself? Probably not, because it requires the pstn and there's no way for a vetting platform to know if its call went through the pstn. We may need a way for a cidvv platform to establish a trust relationship with a cloud cidvv platform so the cloud can vet the switched tollfree and communicate that to the cidvv platform. This allows cidvv to vouch for switched tollfree calls. Maybe it doesn't matter for an originator. Maybe the originator just vouches for the DID and the tollfree. It's in nobody's interest to vouch incorrectly. It's important for the vetter to get it right.
  - [ ] Number porting. Especially internationally. In the EU it's possible to port across countries. There's a central ported routing table. I don't think this has any effect on CIDVV
  - [ ] EU toll free or non-geographic like 0800 and 0900.
- [ ] Bob will need a way to "never vouch" certain numbers. Trading floors want immediate ring. Some callerids will be trusted. Callerid, regex, signers, issuers, attestation levels, cnam. There should be a way to bypass the vouching for specific numbers and immediately return a 302 or 404. Maybe there should be a response code specifically for this.
- [ ] No need for encrypted/hashed tokens. They're too easy to brute force. Need to have language in the usage agreement that says something like "By initiating a CIDVV vouching request, the called party is voluntarily sharing their phone number with the calling party. The called party is choosing to disclose this information by participating in the protocol."
- [ ] First release, it might be better to build APIs into the CIDVV platform and distribute a "static frontend" that the user opens locally in their browser. This is html, js, and css to support a react frontend that talks to the CIDVV platform with APIs. Other projects do this and it would streamline the web hosting.
  - [ ] Refine.dev is good for internal admin panels for frontend hosted on a web server
  - [ ] Flask or FastAPI can host the APIs themselves
- [ ] Trading floors need immediate connection and do not want to wait. The CIDVV platform need an exception list so some callerids are not vouched.
