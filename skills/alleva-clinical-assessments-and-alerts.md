---
name: Record clinical assessments and urgent alerts
description: Submit CIWA and COWS withdrawal assessments for a client, manage urgent alerts, and log shift rounds and incident reports.
api: openapi/alleva-rest-api-openapi.yml
generated: '2026-08-06'
method: generated
operations:
  - POST /clients/{id}/assessments/ciwa
  - PATCH /clients/{id}/assessments/ciwa/{ciwaId}
  - POST /clients/{id}/assessments/cows
  - PATCH /clients/{id}/assessments/cows/{cowsId}
  - GET /clients/{id}/urgent-alerts
  - POST /clients/{id}/urgent-alerts
  - POST /clients/{id}/urgent-alerts/read
  - GET /clients/{id}/history
---

# Record clinical assessments and urgent alerts

Grounded on literal `METHOD /path` pairs in `openapi/alleva-rest-api-openapi.yml`.

## Withdrawal assessments

Alleva models the two standard withdrawal scales as first-class endpoints:

- **CIWA-Ar** (alcohol) — `POST /clients/{id}/assessments/ciwa`, corrected via
  `PATCH /clients/{id}/assessments/ciwa/{ciwaId}`. Body shape: the `Ciwa` /
  `ClientAssessmentsCiwa` schemas.
- **COWS** (opioid) — `POST /clients/{id}/assessments/cows`, corrected via
  `PATCH /clients/{id}/assessments/cows/{cowsId}`. Body shape: the `ClientAssessmentsCows` schema.

Both are **create-then-amend**, never replace. Use `PATCH` to correct a scored assessment so the clinical
audit trail survives; there is no delete.

## Urgent alerts

- `GET /clients/{id}/urgent-alerts` — read active alerts.
- `POST /clients/{id}/urgent-alerts` — raise one. `PATCH` the same path to amend.
- `POST /clients/{id}/urgent-alerts/read` — mark acknowledged (the `AlertReadBy` schema records who).

## Surrounding documentation

`GET`/`POST` under the `ShiftRounds` group for rounds, the `IncidentReport` group for incidents, and
`GET /clients/{id}/history` for the client timeline.

## This is PHI

Every operation here reads or writes protected health information for behavioral-health and substance-use
clients — a category with heightened protection. Do not cache responses, do not send them to a third-party
model, and do not log bodies. Alleva holds ONC Certification, SOC 2 Type II and HIPAA controls
(`security/alleva-trust-center.yml`); the obligations flow to the caller too.

## Error handling

No 4xx/5xx response is described for any operation and no idempotency key exists. Treat non-2xx as opaque
and re-read before retrying a write.
