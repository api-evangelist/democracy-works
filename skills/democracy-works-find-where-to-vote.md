---
name: democracy-works-find-where-to-vote
description: >-
  Find a voter's polling place, early-voting sites and ballot drop boxes from their street
  address, with hours, date ranges and source attribution, using the Democracy Works
  voting-locations endpoint.
api: Democracy Works Voting Locations API
version: '2.0'
base_url: https://api.democracy.works/v2
auth: 'X-API-KEY header'
operations:
  - getVotingLocations
generated: '2026-09-07'
method: generated
source: >-
  Grounded in openapi/_original/democracy-works-api-v2-openapi.json (harvested 2026-09-07
  from https://developers.democracy.works/api/v2)
---

# Find where a voter votes

This operation needs a **real street address**. A ZIP code, a city or a state will not
resolve to a polling place. Ask the user for the address they are registered at, and do not
persist it.

## Call

```
GET /voting-locations
X-API-KEY: <key>
```

Address, one of two ways — never mixed:

- Single string: `address=20 Jay Street Brooklyn NY 11201`
- Components (preferred, and **all four are required together**): `addressStreet`,
  `addressCity`, `addressState`, `addressZip`, plus optional `addressUnitNumber`

Note the parameter is `addressState` here, while `getElections` uses `addressStateCode`.
The two surfaces do not share naming.

`publishStatus` controls whether prepublished data is included.

```bash
curl --get \
  --header "X-API-KEY: $YOUR_API_KEY" \
  --data-urlencode addressStreet="20 Jay Street" \
  --data-urlencode addressCity=Brooklyn \
  --data-urlencode addressState=NY \
  --data-urlencode addressZip=11201 \
  https://api.democracy.works/v2/voting-locations
```

## Read the response

- `votingLocation` — `address`, `latitude`, `longitude`, `pollingHours`, `startDate`,
  `endDate`, `notes`, `sources`
- `normalizedAddress` — what the service resolved your input to. **Show this back to the
  user.** If it normalized to the wrong place, everything downstream is wrong.
- `votingLocationElection` — which election these locations serve (`name`, `electionDay`,
  `ocdDivisionId`)
- `state` — the state and its election administration body

`startDate` and `endDate` matter: early-voting sites and drop boxes are open for a window,
not on election day only. Do not present an early-voting site as an election-day polling
place.

Always surface `sources` and link the state or local election office. This data comes from
official government sources and the attribution is the point.

## Errors — this operation has its own set

| Status | Meaning | Action |
| --- | --- | --- |
| 400 | Bad Request — malformed or partial address components | Send all four components, or the single string |
| 403 | Forbidden — missing/invalid key | Check `X-API-KEY` |
| 422 | Unprocessable Entity — address accepted but not resolvable | Re-prompt for a complete street address |
| 429 | Too Many Requests | Backoff with jitter; no `Retry-After` is sent |
| 500 | Internal Server Error | Retry with backoff |
| 503 | Service Unavailable | Retry; check https://status.democracy.works |

Errors here use the `votingLocationError` shape — a bare `{message}` — not the
`{status, message[]}` envelope the rest of the API uses.

## When you cannot answer

If the address does not resolve, do not guess a nearby location. Point the user at their
state's official polling-place lookup — `pollingLocationUrl` on the state authority from
`getStateAuthorities` — or at TurboVote.
