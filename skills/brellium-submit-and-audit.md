---
name: Submit a clinical document and retrieve its audit
description: Authenticate, upload a clinical note to Brellium, and retrieve the resulting compliance audit.
api: openapi/brellium-openapi-original.json
operations:
  - POST /auth
  - POST /documents
  - GET /documents/{document_id}
  - POST /audits
  - GET /audits/{audit_id}
generated: '2026-07-18'
method: generated
---

# Submit a clinical document and retrieve its audit

Brellium audits clinical documentation against a question set (payor / quality / billing rules)
and returns a scored result. This skill covers the happy path.

## Prerequisites
- A `client_id` and `client_secret` (obtained from a Brellium account manager / support@brellium.com).
- The target `set_id` (question set) — list them with `GET /question-sets`.

## Steps
1. **Authenticate.** `POST /auth` with `{ "client_id": ..., "client_secret": ... }`. Store the
   returned JWT. Send it as `Authorization: Bearer <JWT>` on every subsequent call. Tokens expire
   after 24h — on a `400`, refresh the token via `/auth` and retry (per Brellium's guidance).
2. **Upload the note.** `POST /documents` (JSON body, or multipart with a `file`). Attach your own
   identifiers via the `external_metadata` object. For large files, first call
   `POST /documents/upload-url` to get a presigned URL. For a note spanning multiple files, use
   `POST /documents-multiple`. The response returns a `document_id`.
3. **Poll document status.** `GET /documents/{document_id}` until `status` indicates the audit is
   created (`audit_created: true`).
4. **Retrieve the audit.** `POST /audits` with an `AuditFilter` body (filter by `set_id`, date
   ranges, `scoreLowerBound/UpperBound`; paginate with `page` + `pageSize` up to 1000), or fetch a
   known result directly with `GET /audits/{audit_id}`. Set `includeDisplayInfo` / `includePeople` /
   `includeQuestionIds` to expand the response.

## Conventions & errors
- Auth: JWT bearer (see `conventions/brellium-conventions.yml`).
- Pagination: `page` + `pageSize` (max 1000).
- Errors: plain JSON `{ message }`; `401` = re-authenticate (see `errors/brellium-problem-types.yml`).
- No idempotency-key contract is documented — do not assume safe retries on writes.
