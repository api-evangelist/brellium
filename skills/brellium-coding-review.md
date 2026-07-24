---
name: Review coding (E&M / CPT / ICD) evaluations
description: Authenticate and query Brellium coding evaluations to find upcoded/downcoded charts and CPT/ICD misalignment.
api: openapi/brellium-openapi-original.json
operations:
  - POST /auth
  - POST /coding
  - GET /coding/{coding_eval_id}
generated: '2026-07-18'
method: generated
---

# Review coding (E&M / CPT / ICD) evaluations

Brellium's Coding Module evaluates charts for E&M / CPT / ICD accuracy and flags upcoding,
downcoding, and code misalignment. This skill queries those evaluations.

## Steps
1. **Authenticate.** `POST /auth` → JWT bearer token (24h TTL). Send `Authorization: Bearer <JWT>`.
2. **Query coding evaluations.** `POST /coding` with a `CodingEvalFilter` body. Useful filters:
   `emScoreLowerBound`/`emScoreUpperBound`, `cptCodes`, `recommendedCptCodes`, `icdCodes`,
   `hccCodes`, `upcoded`, `downcoded`, `baseCodeAligned`, `mdmLevel`/`riskLevel`/`dataLevel`, plus
   session/eval/last-updated date ranges. Paginate with `page` + `pageSize`. Expand with
   `includeDisplayInfo` / `includePeople` / `includeResponses`.
3. **Drill into one evaluation.** `GET /coding/{coding_eval_id}` for the full `em_eval`, `icd_eval`,
   linked `documents`, and criteria `responses`.

## Conventions & errors
- Auth + pagination + error envelope: see `conventions/brellium-conventions.yml` and
  `errors/brellium-problem-types.yml`.
- A `401` means the token expired — re-run `POST /auth` and retry.
