---
name: Create and send a TRATO contract for signature
description: >-
  Create a contract in TRATO from a template or uploaded document, set
  participant variables, send it for electronic signature, and retrieve the
  signed documents once complete.
api: openapi/trato-openapi.yml
operations:
  - createContractV2
  - listTemplates
  - setParticipantVariables
  - sendContract
  - getContractStatus
  - getSignedDocuments
generated: '2026-07-21'
method: generated
source: https://developer.trato.io/
---

# Create and send a TRATO contract for signature

TRATO is an AI-powered Contract Lifecycle Management (CLM) and e-signature
platform. All API calls go to `https://enterprise.api.trato.io` and require a
JWT bearer token in the `Authorization: Bearer {TOKEN}` header (get the token
from your profile security settings). Responses carry a boolean `success` field.

## Steps

1. **(Optional) Pick a template.** Call `listTemplates`
   (`GET /api/list/templates`) to find a `templateId`. You can filter with
   `name`, `dateStart`, and `dateEnd`.

2. **Create the contract.** Call `createContractV2`
   (`POST /api/v2/create/contract`) with the source template or an uploaded
   PDF/WORD document. The response returns `{ "success": true, "contractid": "..." }`.
   Keep the `contractid` for every later call.

3. **Set participant variables.** Call `setParticipantVariables`
   (`POST /api/contract/variables/{contractID}/{participantID}`) to fill each
   signer's fields.

4. **Send for signature.** Call `sendContract`
   (`POST /api/contract/send/{contractID}`) to dispatch the contract to
   participants for electronic signing.

5. **Track status.** Poll `getContractStatus`
   (`GET /api/contract/status/{contractID}`) — the `status` moves through states
   such as `AUTHORIZE` → signed → `FINALIZED`. Prefer subscribing to the
   `signed`, `signed-all`, and `document-finalized` webhooks (verified via the
   `X-Trato-Secret` header) instead of polling.

6. **Retrieve signed documents.** Once finalized, call `getSignedDocuments`
   (`POST /api/contract/documents/{contractID}`) to download the executed files.

## Notes

- TRATO does not document an idempotency key — use your own `externalId` as a
  client reference (it is echoed on status and webhook payloads) and avoid blind
  retries of `createContractV2`.
- For Mexican legal validity you can call `validateNom151`
  (`POST /api/contracts/{contractID}/validate/nom151`).
- Errors return `{ "success": false, "message": "..." }`; a 401 means the bearer
  token is missing or invalid.
