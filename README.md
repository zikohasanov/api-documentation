# Pickup Ordering API Documentation

> Portfolio API contract for a fictional self-initiated case study. It is not a live service, production API, or client deliverable.

The contract is intentionally small: menu discovery, idempotent pickup-order submission, secure status lookup, and staff-controlled state transition. [OpenAPI 3.1 contract](openapi.yaml) is the machine-readable source; [API guide](api-guide.md) explains behavior, errors, security assumptions, and unresolved deployment decisions.

## Contract principles

- Submission creates a request in `awaiting_review`; it does not promise fulfilment.
- A client-supplied idempotency key protects retries from duplicate orders.
- Prices and selected options are snapshotted on creation; server-side totals are authoritative.
- Customer status access uses a one-time verification challenge; staff operations require a staff bearer credential and role authorization.
- Error responses use a consistent problem-details shape and must not disclose whether an arbitrary reference belongs to a real order.

The paths and schemas are proposed design artifacts only. No endpoint is available to call.
