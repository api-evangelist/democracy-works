---
name: democracy-works-find-upcoming-elections
description: >-
  Find every upcoming election a US voter is eligible to vote in, from an address or a
  state, using the Democracy Works Elections API — with the dates, deadlines and
  registration/voting methods that apply to each one.
api: Democracy Works Elections API
version: '2.0'
base_url: https://api.democracy.works/v2
auth: 'X-API-KEY header'
operations:
  - getElections
generated: '2026-09-07'
method: generated
source: >-
  Grounded in openapi/_original/democracy-works-api-v2-openapi.json (harvested 2026-09-07
  from https://developers.democracy.works/api/v2) and the provider's own guide
  "Integrating Election Data into AI Experiences", linked from
  https://www.democracy.works/search-social-ai
---

# Find a voter's upcoming elections

## Before you call anything

Democracy Works asks integrators to do three things, and they are not optional courtesies:

1. **Ask for location first.** Election rules vary by state and by locality. Ask for the
   state at minimum. For local elections and polling places you need the full street
   address — a ZIP code is not enough.
2. **Cite the source.** Return the `canonicalUrl` the API gives you alongside any answer.
3. **Decline when there is no data.** If the API returns nothing relevant, say so and point
   the user at their Secretary of State site or TurboVote. Do not answer from memory —
   deadlines change and a wrong deadline disenfranchises someone.

Cache responses for **no more than one hour** (the provider's stated ceiling).

## Step 1 — call `getElections`

`GET /elections` with `X-API-KEY: <key>`.

Supply the location one of two ways. Do not mix them:

- Single string: `address=813 Howard Street Oswego NY 13126`
- Components (preferred): `addressStreet`, `addressCity`, `addressStateCode`, `addressZip`
  (and optionally `addressZip4`)

Or skip the address entirely and pass `stateCode` / `ocdId` for a broader view.

Narrow the window with `startDate` (omit it and you get past elections too) and `endDate`,
and filter with `electionTypes`.

```
GET /elections?addressStreet=813%20Howard%20Street&addressCity=Oswego&addressStateCode=NY&addressZip=13126&startDate=2026-09-07
X-API-KEY: <key>
```

## Step 2 — read `guidancePhase` before you present anything

Every election carries a phase, and it changes what you are allowed to imply:

- `active` — details vetted and confirmed. Safe to state deadlines.
- `known` — the election is happening but key details and deadlines are **not yet
  released**. Say the election exists; do not present its fields as final.
- `past` — the election date has gone by.

## Step 3 — read the fields that answer the question

`date`, `description`, `type`, `registration` (methods and deadlines), `voting` (by-mail,
early, in-person), `pollingLocationUrl`, `website`, `canonicalUrl`.

For Q&A guidance content, re-call with `includeQuestionAndAnswer=true`. For contests and
ballot measures, re-call with `includeBallotData=true` — they are omitted by default.

## Step 4 — control the response size

Large queries fail with **413 `Response size too large. Try again with a smaller pageSize.`**
This is not a rate limit and waiting will not help. Either:

- lower `pageSize` (default 10, max 100) and page with `page`, reading `pagination`
  (`totalRecordCount`, `currentPage`, `pageSize`); or
- pass a `fields` mask — protobuf FieldMask path syntax, e.g.
  `fields="ocdId,date,registration.deadline,canonicalUrl"`.

Reach for the field mask first: it is cheaper for both sides.

## Language

Pass `Accept-Language: es` (or `en`, `en-US`, `es-US`) for Spanish guidance. Unlocalized
fields come back **null**, not English — handle the null rather than assuming a fallback.

## Errors

| Status | What it means | What to do |
| --- | --- | --- |
| 400 | Bad parameter, or an incomplete address component set | Fix the parameters; send all four address components or the single string |
| 403 | Missing or invalid key (indistinguishable) | Check the `X-API-KEY` header |
| 413 | Response too large | Smaller `pageSize` or a `fields` mask |
| 429 | Throttled | Exponential backoff with jitter — there is no `Retry-After` and no rate-limit headers |
| 500 | Server error | Retry with backoff; check https://status.democracy.works |

Error bodies are **not** RFC 9457. Application errors are `{status, message[]}`; gateway
errors (403/429) are `{message}`. Handle both.
