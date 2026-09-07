---
name: democracy-works-explore-the-ballot
description: >-
  Walk a voter through what is actually on their ballot — contests, candidates, ballot
  measures and endorsements — starting from an election and drilling into each object by id,
  using the Democracy Works Elections API.
api: Democracy Works Elections API
version: '2.0'
base_url: https://api.democracy.works/v2
auth: 'X-API-KEY header'
operations:
  - getElections
  - getContest
  - getCandidate
  - getBallotMeasure
  - getEndorsement
  - getEndorsementBulk
generated: '2026-09-07'
method: generated
source: >-
  Grounded in openapi/_original/democracy-works-api-v2-openapi.json (harvested 2026-09-07
  from https://developers.democracy.works/api/v2)
---

# Explore what is on the ballot

Ballot content is **not** in the default election response. You have to ask for it, and then
drill in by id.

## Step 1 — get the election with ballot data

```
GET /elections?stateCode=PA&startDate=2026-09-07&includeBallotData=true
X-API-KEY: <key>
```

Ballot measure coverage is documented for all federal elections and all statewide
elections. Contest and candidate coverage is broader but not universal — local coverage
depends on the jurisdiction. Ballot data is sourced from Ballotpedia.

The election now carries `contests[]` and `ballotMeasures[]`.

## Step 2 — drill into a contest

```
GET /contests?id=<contestId>
```

Fields that change how you must present it:

- `contestType`, `level`, `branch`, `districtName`, `districtType` — what office this is
- `seatsUpForElection` — "vote for up to N", not "vote for one"
- `rankedChoice`, `rankedChoiceRankNumber`, `rankedChoiceExplainerURL` — if true, explain
  ranking before listing candidates
- `hasPrimary`, `primaryDate`, `generalDate`, `partisanPrimary`,
  `partisanPrimaryExplainerEn` / `partisanPrimaryExplainerEs` — whether a voter's party
  registration restricts which primary ballot they may take. Getting this wrong is a
  documented way to disenfranchise someone.
- `cancelled` — **check this first**. A cancelled contest must never be presented as live.
- `candidates[]` — embedded

## Step 3 — drill into a candidate

```
GET /candidates?id=<candidateId>
```

`fullName`, `partyAffiliation`, `isIncumbent`, `isWriteIn`, `status`, `ballotpediaUrl`,
`runningMateFullName` / `runningMateTitle`, `rankedChoiceVotingRound`, `endorsementCount`.

`isWriteIn` and `status` matter: a withdrawn or write-in candidate is not the same as a
candidate printed on the ballot.

## Step 4 — drill into a ballot measure

```
GET /ballot-measures?id=<ballotMeasureId>
```

`ballotQuestion` is the text as it appears on the ballot. `summary`, `yesVote` and `noVote`
explain what each vote does — quote these rather than paraphrasing, because paraphrasing a
ballot question is how a neutral measure becomes a persuasive one. `topics` / `topicAreas`
categorize it; `status` says where it stands; `yesVotesTotal` / `noVotesTotal` carry results
once counted.

## Step 5 — endorsements

```
GET /endorsements?id=<endorsementId>
GET /endorsements/bulk/byMatchingEntity?candidateId=<id>
GET /endorsements/bulk/byMatchingEntity?ballotMeasureId=<id>
```

Endorsements are `{id, type, name, source, position}`. **Always attribute** — name the
endorsing organization and its `source`. An endorsement presented without its source reads
as the tool's own recommendation, which is exactly what a nonpartisan integration must not
do.

`candidate.endorsementCount` and `ballotMeasure.endorsementYesCount` /
`endorsementNoCount` tell you whether a bulk call is worth making.

## Guardrails

- Present contests and measures neutrally and completely. Omitting a candidate or a measure
  is itself a distortion.
- Never rank, score or recommend. Report endorsements as facts with attribution.
- Cite `canonicalUrl` and link the official election office.
- Cache for at most one hour.
