# API Guide

## Environment and authentication

No live base URL is provided because this is a contract-only portfolio exercise. A deployed implementation would serve HTTPS only. Public menu reads require no account. Staff order transitions require an authenticated staff identity with an appropriate role. Customer status lookup requires an order reference plus a short-lived, one-time verification code delivered to the contact channel collected at checkout.

Secrets, real phone numbers, and customer records must not be included in repository examples. The verification code is a placeholder flow: provider choice, consent wording, expiry, delivery retry, rate limits, and local privacy requirements remain deployment decisions.

## Submit an order

`POST /v1/orders` accepts item identifiers, quantities, selected option identifiers, a customer name, contact channel/value, and acknowledgements. Send a unique `Idempotency-Key` for each deliberate submission. Reuse the same key only when retrying the same logical request. The server revalidates menu data and computes prices; client totals are not trusted.

Success returns `202 Accepted`, a reference, `awaiting_review`, and a status-check instruction. `202` means the service accepted the request for processing, not that restaurant staff accepted fulfilment.

## Check status

`POST /v1/orders/{orderReference}/verification` requests a one-time code without returning order details. `POST /v1/orders/{orderReference}/status` validates the code and returns only the current state, last update time, and pickup guidance. A wrong, expired, or rate-limited challenge returns a generic error. The reference alone is not an authorization credential.

## Staff transition

`POST /v1/staff/orders/{orderReference}/transitions` requires staff authentication, a target state, and an expected current state. The server enforces the transition graph and records the actor and time atomically. A conflict returns `409` with the current state, allowing the interface to refresh instead of overwriting another user's action.

## Errors and retries

Errors use `application/problem+json` with `type`, `title`, `status`, `detail`, and optional field `errors`. Clients may retry transient `5xx` and network failures using the same idempotency key. Do not automatically retry validation, authorization, or state-conflict errors without user or staff review.

## Open questions for implementation

- Which notification provider and verification channel will be used?
- What are rate limits, code expiry, and data-retention duration?
- Which currency, tax rules, and configured charges apply?
- What staff roles can reject or cancel an order?
- What is the order-review timeout and escalation policy?

## State representation

The business-analysis artifacts use title case for readability (`AwaitingReview`, `Preparing`); this API serializes the same states in `snake_case` (`awaiting_review`, `preparing`). The transition order and terminal states are otherwise identical across the portfolio.

## Idempotency-key behavior

The server should bind each idempotency key to a request fingerprint for a documented retention window. A retry with the same key and same logical request returns the original result; reuse of that key with a materially different request returns `409 Conflict`. A new deliberate order uses a new key.
