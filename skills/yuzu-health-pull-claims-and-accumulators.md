---
name: Pull claims and coverage accumulators
description: Retrieve medical and pharmacy claims and read a coverage's benefit accumulators on Yuzu Health.
api: openapi/yuzu-health-openapi-original.json
operations:
  - PublicApiV1Controller_listAllMedicalClaims
  - PublicApiV1Controller_getMedicalClaim
  - PublicApiV1Controller_listAllPharmacyClaims
  - PublicAccumulatorController_getCoverageAccums
  - PublicAccumulatorController_getCoverageBuckets
---

# Pull claims and coverage accumulators

Retrieve adjudicated claims and read where a member stands against their benefit accumulators.

## Auth
`Authorization: Bearer <API_KEY>` (portal Settings > Developer). Base URL `https://api.yuzu.health`.

## Steps
1. **List medical claims** — `GET /v1/claims/medical`
   (`PublicApiV1Controller_listAllMedicalClaims`); page with `limit` (default 50) + `startAfter`.
   Fetch one with `GET /v1/claims/medical/{claimId}` (`PublicApiV1Controller_getMedicalClaim`).
2. **List pharmacy claims** — `GET /v1/claims/pharmacy`
   (`PublicApiV1Controller_listAllPharmacyClaims`).
3. **Read accumulators** — for a coverage, `GET /v2/accumulator/coverage/{coverageId}`
   (`PublicAccumulatorController_getCoverageAccums`) and
   `GET /v2/accumulator/coverage/buckets/{coverageId}`
   (`PublicAccumulatorController_getCoverageBuckets`) for deductible/OOP bucket ranges.
4. **Download an EOB** — `GET /v2/eob/download/{claimId}` (`PublicEobController_downloadEob`) returns
   the EOB PDF for a claim.

## Rules
- Results are ordered deterministically by `createdAt` then `id`; keep paging while a full page
  (`limit` items) comes back, passing the last `id` as `startAfter`.
- A `404` on a claim/coverage means it isn't owned by your API key's org. `400` returns a Zod
  `errors[]` array. Accumulator and EOB endpoints are read-only (`connected` agent access).
