---
name: democracy-works-explain-state-voting-rules
description: >-
  Answer "how do I register to vote" and "how do I vote" for a US state with authoritative,
  current instructions and the official election-office links, using the Democracy Works
  Authorities endpoints.
api: Democracy Works Authorities API
version: '2.0'
base_url: https://api.democracy.works/v2
auth: 'X-API-KEY header'
operations:
  - getStateAuthorities
  - getAuthorities
  - getLocalAuthorities
generated: '2026-09-07'
method: generated
source: >-
  Grounded in openapi/_original/democracy-works-api-v2-openapi.json (harvested 2026-09-07
  from https://developers.democracy.works/api/v2) and the provider's guide "Integrating
  Election Data into AI Experiences"
---

# Explain how to register and vote in a state

Authority data is **evergreen**: statewide instructions that do not change election to
election. This is the right endpoint for "how do I register in Ohio", and the wrong one for
"when is my next election" (that is `getElections`).

## Step 1 — get the state authority

```
GET /authorities/state/{stateCode}
X-API-KEY: <key>
```

`stateCode` is the two-letter postal code; DC is accepted. Omit it and you get all 51
top-level authorities. Add `includeQuestionAndAnswer=true` for the guidance Q&A content.

## Step 2 — use the fields that actually answer the question

- `registration` — available methods (online, by-mail, in-person, election-day) and youth
  pre-registration via `youthRegistration`
- `voting` — by-mail, early and in-person voting methods
- `homepageUrl` — the state's official election site. This is the link to give a user who
  needs to go further, and it is the one the provider's own minimum-viable guidance names:
  `data['authorities'][0]['homepageUrl']`.
- `localElectionAuthorityLookupUrl` / `localRegistrationAuthorityLookupUrl` — where a voter
  finds their local office
- `studentVotingUrl`, `votingWithDisabilitiesUrl`, `votingWithPastConvictionsUrl` — the
  specific-population pages; reach for these when the question calls for one
- `contact` — the office's real contact details
- `canonicalUrl` — cite this in your answer
- `timezone` / `secondaryTimezone` — a deadline is meaningless without the zone

`separateRegistrationAndElectionAuthorities` tells you whether registration and voting are
run by different offices in that state. If true, do not conflate the two contacts.

## Step 3 — go local only when the question is local

```
GET /authorities/local?address=...        # or ?stateCode=XX
GET /authorities?address=...              # or ?stateCode=XX
```

Local authorities return a thinner shape (`officeName`, `officialTitle`, `homepageUrl`,
`contact`, and the authority-level flags).

## Identifiers

Every authority is keyed on an **OCD-ID** and is uniquely identifiable by it —
`ocd-division/country:us/state:ny/county:albany`, and
`ocd-division/country:us/district:dc` for DC. Use it to join authority data to election
data. (Elections carry an OCD-ID too but are **not** uniquely identified by it.)

## Guardrails

- Never state a rule the response does not contain. Voter-ID requirements, felony
  disenfranchisement rules and pre-registration ages are exactly the places where a
  confident guess causes harm.
- Prefer linking the state's own page over paraphrasing it.
- Cache for at most one hour, per the provider's guidance.
