---
name: Authenticate and ingest transactions
description: Obtain a scoped Cable access token, then push single and batch transactions into Cable for automated compliance testing.
api: openapi/cable-api-reference-openapi-original.yml
operations: [request-token, add-transaction, add-transactions-batch, check-transaction, update-transaction]
---

# Authenticate and ingest transactions into Cable

Cable's API is a data-ingestion entry point: you send it transaction data and Cable
continuously tests it against your compliance controls. Base URL: `https://api.cable.tech`.

## Steps

1. **Get an access token** — `request-token` (`POST /v2/auth/token`). Send your Cable-issued
   refresh/auth key in the `Authorization` header. Body: `expiry_seconds` (300-86400) and
   `scopes` = `{ organization_id, scopes: ["transactions:write"] }`. You get back a short-lived
   bearer `token`. You cannot request a new token using an existing access token.
2. **Ingest one transaction** — `add-transaction` (`POST /v2/transaction`) with the bearer token.
   Provide `tx_id`, `timestamp`, `type`, `method`, `amount`, `currency`, `sender_id`,
   `receiver_id`, `status`, and `is_test` for test-mode data.
3. **Ingest in bulk** — `add-transactions-batch` (`POST /v2/transactions/batch`) to submit many
   transactions in one call; prefer this for high throughput to stay under rate limits.
4. **Check before/after** — `check-transaction` (`GET /v2/transaction`) confirms whether a
   transaction id already exists.
5. **Correct data** — `update-transaction` (`PUT /v2/transaction`) to update an already-ingested
   transaction.

## Rules

- **Auth:** bearer token in `Authorization`; refresh via `request-token` when expired (401).
- **Idempotency:** there is no Idempotency-Key. A duplicate `tx_id` returns **409**; use
  `check-transaction` first or switch to `update-transaction`.
- **Errors:** custom `GeneralError { code, message, errors[] }` JSON (not RFC 9457). 400 =
  validation (inspect `errors[]`), 401 = token, 429 = rate limited (back off / batch).
- See `conventions/cable-conventions.yml` and `errors/cable-error-codes.yml`.
