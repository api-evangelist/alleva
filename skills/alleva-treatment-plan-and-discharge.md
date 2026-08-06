---
name: Read treatment plans, capture signatures, and reach discharge
description: Pull a client's treatment plan and diagnoses, capture plan and review signatures, read treatment reviews, and retrieve the discharge plan.
api: openapi/alleva-rest-api-openapi.yml
generated: '2026-08-06'
method: generated
operations:
  - GET /treatment-plans
  - GET /treatment-plans/{id}
  - GET /treatment-plans/{id}/diagnosis
  - POST /treatment-plans/{id}/signature
  - POST /treatment-plans/{id}/review-signature
  - GET /treatment-reviews
  - GET /treatment-reviews/{id}
  - GET /discharge-plan
  - GET /discharge-plan/{id}
---

# Read treatment plans, capture signatures, and reach discharge

Grounded on literal `METHOD /path` pairs in `openapi/alleva-rest-api-openapi.yml`.

## Read the plan

1. `GET /treatment-plans` — list, with `Cursor` + `Limit`, `StartDate`/`EndDate`, and `fields`.
2. `GET /treatment-plans/{id}` — the plan itself (`TreatmentPlan`, `BehavioralDefinition` and related
   schemas; see `data-model/alleva-data-model.yml`).
3. `GET /treatment-plans/{id}/diagnosis` — the attached diagnoses.

## Signatures

- `POST /treatment-plans/{id}/signature` — the plan signature.
- `POST /treatment-plans/{id}/review-signature` — the review signature.

These are **legally meaningful clinical attestations**. Never call them on an agent's own initiative;
require an explicit, in-the-moment human authorization from the signing clinician for each call. There is
no revoke endpoint and no idempotency key — a duplicate POST is a second attestation you cannot withdraw
through the API.

## Reviews and discharge

- `GET /treatment-reviews` / `GET /treatment-reviews/{id}` — the utilization-review record.
- `GET /discharge-plan` / `GET /discharge-plan/{id}` — the discharge plan. Both are **read-only** on this
  API; discharge planning is authored in the product, not through the contract.
- `GET /levels-of-care` — level-of-care definitions the plan references; writable via
  `POST`/`PATCH /levels-of-care`.

## Error handling

Only 200 responses are described. Treat any other status as opaque, and never auto-retry a signature call.
