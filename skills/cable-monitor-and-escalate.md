---
name: Ingest checks, alerts, and suspicious activity
description: Push transaction-monitoring outcomes into Cable — checks, alerts, suspicious activities, and customer SARs — so Cable can assurance-test the monitoring chain.
api: openapi/cable-api-reference-openapi-original.yml
operations: [add-transaction-check, add-transaction-alert, add-transaction-suspicious-activity, add-customer-sar]
---

# Ingest the transaction-monitoring and escalation chain

Cable tests the full financial-crime monitoring lifecycle. After transactions are ingested,
send the downstream monitoring artifacts so Cable can assess whether controls fired correctly.
Authenticate first with `request-token` (see the transactions skill). Base URL:
`https://api.cable.tech`.

## Steps

1. **Ingest checks** — `add-transaction-check` (`POST /v2/transaction_check`). Each check relates
   to a transaction via `related_tx_id`.
2. **Ingest alerts** — `add-transaction-alert` (`POST /v2/transaction_alert`). An alert relates
   to a check via `related_check_id` and to transactions via `related_tx_ids`.
3. **Ingest suspicious activity** — `add-transaction-suspicious-activity`
   (`POST /v2/transaction_suspicious_activity`), linked to an alert via `related_alert_id` and to
   a `related_person_id` / `related_company_id`.
4. **Ingest a customer SAR** — `add-customer-sar` (`POST /v2/customer_sar`) when a Suspicious
   Activity Report is filed for a person or company.

## Rules

- **Order matters:** ingest transactions, then checks, then alerts, then suspicious activity,
  then SARs — each references ids from the prior stage (see `data-model/cable-data-model.yml`).
- **Auth / errors / idempotency:** same contract as the transactions skill — bearer token, 409 on
  duplicate ids, custom `GeneralError` envelope, 429 back-off. Batch/`PUT` update variants exist
  for each resource.
