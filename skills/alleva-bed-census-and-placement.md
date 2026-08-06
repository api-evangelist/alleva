---
name: Manage bed census, reservations and placement
description: Read facility bed inventory, reserve a bed for a lead, convert the reservation to an occupancy, transfer, and end a stay.
api: openapi/alleva-rest-api-openapi.yml
generated: '2026-08-06'
method: generated
operations:
  - GET /facilities
  - GET /rooms
  - GET /beds
  - POST /bed/{bedId}/reservations
  - GET /bed/-/reservations
  - POST /bed/{bedId}/reservations/occupy
  - GET /bed/-/occupancies/occupants
  - POST /bed/{bedId}/occupancies/transfer
  - DELETE /bed/{bedId}/occupancies/end/{id}
---

# Manage bed census, reservations and placement

Grounded on literal `METHOD /path` pairs in `openapi/alleva-rest-api-openapi.yml`. Alleva declares no
`operationId` anywhere, so there is none to cite.

## The `-` wildcard

Every bed route is published twice: `/bed/{bedId}/…` scoped to one bed, and `/bed/-/…` where `-` is a
literal path segment meaning *across all beds*. Use `/bed/-/…` for census-wide reads and
`/bed/{bedId}/…` for a specific placement.

## Steps

1. **Inventory** — `GET /facilities`, `GET /rooms`, `GET /beds`. Filter beds with the `facilityId` query
   parameter where offered.
2. **Current census** — `GET /bed/-/occupancies/occupants` for who is in a bed right now, and
   `GET /bed/-/reservations/occupants` for who is holding one.
3. **Reserve** — `POST /bed/{bedId}/reservations` for a lead. Adjust with
   `PATCH /bed/{bedId}/reservations/{id}`; release with `DELETE /bed/{bedId}/reservations/{id}` or
   `DELETE /bed/{bedId}/reservations/remove-lead/{leadId}`.
4. **Admit into the bed** — `POST /bed/{bedId}/reservations/occupy` converts the hold into an occupancy.
   `POST /bed/{bedId}/occupancies` creates one directly when there was no reservation.
5. **Transfer** — `POST /bed/{bedId}/occupancies/transfer` to move a client between beds. Do not end and
   recreate; the transfer endpoint exists so the census history stays intact.
6. **Discharge from the bed** — `DELETE /bed/{bedId}/occupancies/end/{id}`, or
   `DELETE /bed/{bedId}/occupancies/end-lead/{leadId}` when you hold the lead rather than the occupancy id.

## Pagination

List reads accept `Cursor` + `Limit`, plus `StartDate`/`EndDate` and a `fields` sparse-fieldset parameter.
The response envelope is not described in the spec, so read the first page and inspect the body to learn
the next-cursor field name.

## Warning

`end`, `transfer` and `remove-lead` are destructive census writes with **no idempotency key**. Re-read
with `GET /bed/-/occupancies/occupants` after any failure instead of retrying.
