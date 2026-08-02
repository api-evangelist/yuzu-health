---
name: Create a group policy and enroll members
description: Create a group, run an initial enrollment, then list the enrolled members on Yuzu Health.
api: openapi/yuzu-health-openapi-original.json
operations:
  - PublicApiV2Controller_createGroup
  - PublicApiV2Controller_createInitialEnrollment
  - PublicApiV2Controller_listGroupMembers
  - PublicEnrollmentApplicationController_createAddSubscriber
---

# Create a group policy and enroll members

Use the Yuzu Health API to stand up a self-funded group and enroll its members.

## Auth
All calls use `Authorization: Bearer <API_KEY>`. Generate the key in the vendor portal under
Settings > Developer. Production base URL is `https://api.yuzu.health`; test against
`https://backend.yuzu.earth`.

## Steps
1. **Create the group** — `POST /v2/groups` (`PublicApiV2Controller_createGroup`) with a
   `GroupCreationDto`. Capture the returned `groupPolicyId`.
2. **Run initial enrollment** — `POST /v2/groups/{groupPolicyId}/enroll/initial-enrollment`
   (`PublicApiV2Controller_createInitialEnrollment`) with an `InitialEnrollmentDto`. This returns an
   enrollment request; poll `POST /v2/enroll/{enrollmentRequestId}/status`
   (`PublicApiV2Controller_getEnrollmentRequestStatus`) until it completes.
3. **Add later subscribers** — for members joining after go-live, use
   `POST /v3/groups/{groupPolicyId}/enroll/add-subscriber`
   (`PublicEnrollmentApplicationController_createAddSubscriber`); add dependents with
   `POST /v3/groups/{groupPolicyId}/enroll/{subscriberCoverageId}/add-dependent`.
4. **Verify** — `GET /v2/groups/{groupPolicyId}/members`
   (`PublicApiV2Controller_listGroupMembers`), paging with `limit` (default 100 for member lists) and
   `startAfter`.

## Rules
- No idempotency key is supported; do not blindly retry a `POST` enroll on timeout — first re-list
  members / re-check enrollment status to avoid duplicate enrollment.
- Errors return `{timestamp, path, message}`; a `400` adds a Zod `errors[]` array naming the invalid
  fields. A `404` means the group/coverage isn't visible to your key's owning org.
