---
name: democracy-works-pull-bulk-exports
description: >-
  Retrieve a contracted Democracy Works bulk data export as a short-lived presigned S3 URL,
  and handle it safely — the URL is a credential.
api: Democracy Works Exports API
version: '2.0'
base_url: https://api.democracy.works/v2
auth: 'X-API-KEY header'
operations:
  - getExports
generated: '2026-09-07'
method: generated
source: >-
  Grounded in openapi/_original/democracy-works-api-v2-openapi.json (harvested 2026-09-07
  from https://developers.democracy.works/api/v2)
---

# Pull a bulk data export

For partners with a bulk export arranged. If none is configured, this returns nothing —
setting one up is a conversation with partnerships@democracy.works, not an API call.

## Call

```
GET /exports
X-API-KEY: <key>
```

No parameters. The operation takes none.

## Response

An array of `{exportName, exportUrl}`. `exportUrl` is an **AWS presigned S3 URL**.

## Handle the URL as a secret

The contract is explicit, and this is the part an automated caller gets wrong:

- **It expires after one hour.** Download promptly. If it expires, call `/exports` again for
  a fresh URL — there is no refresh, renew or extend operation.
- **It grants access to your data.** Do not share it, do not log it, do not put it in a
  ticket, do not echo it back to a user, and do not store it anywhere the file itself would
  not belong. The signature in the query string *is* the credential.

## Downloading

Fetch `exportUrl` with a plain HTTP GET and **no** `X-API-KEY` header — the signature
authorizes the request, and sending your API key to S3 leaks it to a third party.

## Errors

`/exports` declares only a 200 in the contract. In practice the surface-wide failures still
apply: 403 without a valid key, 429 when throttled, 500 on a server fault. Retry a 429 with
exponential backoff and jitter; there is no `Retry-After`.

## Reversibility

There is nothing to undo — this is a read. The only time dimension is the one-hour URL
expiry, which is a credential lifetime, not a reversal window.
