---
name: Admit a prospect into Alleva
description: Create a prospect, attach insurance and contacts, then read the resulting client record and its intake sections.
api: openapi/alleva-rest-api-openapi.yml
generated: '2026-08-06'
method: generated
operations:
  - POST /prospects
  - POST /prospects/{id}/insurances
  - POST /prospects/{id}/contacts
  - GET /prospects/{id}
  - GET /intakes/types
  - POST /intakes/{intakeType}/sections/{sectionType}
  - GET /clients/{id}
---

# Admit a prospect into Alleva

Alleva's OpenAPI declares **no `operationId` on any of its 424 operations**, so every step below is
grounded on the literal `METHOD /path` pair that exists in `openapi/alleva-rest-api-openapi.yml`. Verify
each against that file before use; do not invent an operationId.

## Before you start

- Send `Authorization: Bearer <jwt>` on every request. There is no public token endpoint — the credential
  is provisioned by Alleva out of band (see `authentication/alleva-authentication.yml`).
- Pick one versioning mechanism and stay on it: the `/v{version}/` path prefix, the `api-version` query
  parameter, or the `X-Version` header. All three exist in parallel and Alleva documents no precedence
  rule between them.
- **There is no idempotency contract.** No `Idempotency-Key` header exists anywhere in the spec, so a
  retried `POST /prospects` can create a duplicate. Before retrying a write, re-read with
  `GET /prospects` or `GET /clients/duplicates` and reconcile rather than blindly replaying.

## Steps

1. **Create the prospect** — `POST /prospects` with the `Prospect` body. Capture the returned id.
2. **Attach insurance** — `POST /prospects/{id}/insurances`. Correct with `PATCH /prospects/insurances/{id}`.
3. **Attach contacts** — `POST /prospects/{id}/contacts` for each emergency/family contact.
4. **Read it back** — `GET /prospects/{id}` to confirm the record landed before moving on. This is the
   substitute for an idempotency guarantee.
5. **Discover the intake shape** — `GET /intakes/types`, then `GET /intakes/{intakeType}` for the section
   layout that facility uses.
6. **Fill intake sections** — `POST /intakes/{intakeType}/sections/{sectionType}`, then
   `PUT /intakes/{intakeType}/sections/{sectionType}/{intakeId}` to revise.
7. **Read the client record** — `GET /clients/{id}`; add `GET /clients/{id}/contacts` and
   `GET /clients/{id}/history` for the full picture.

## Bulk

Use `POST /prospects/import` for a batch. Then run `GET /clients/duplicates` — Alleva exposes duplicate
detection as a first-class endpoint precisely because imports create them.

## Error handling

The spec declares **only 200 responses** — no 4xx or 5xx is described for any operation, and the live API
returns a bare `401` with an empty body. Treat any non-2xx as opaque: log the status, do not attempt to
parse an error envelope, and never auto-retry a write.
