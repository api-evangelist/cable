---
name: Ingest customer onboarding and KYC data
description: Send person/company records, identity verifications, onboarding flows, screening results, and risk assessments into Cable for KYC/onboarding assurance.
api: openapi/cable-api-reference-openapi-original.yml
operations: [add-person-info, add-company-info, add-identity-verification, add-onboarding-flow, add-screening, add-risk-assessment]
---

# Ingest customer onboarding and KYC data

Cable assurance-tests customer onboarding and KYC/KYB controls. Send the entity records and the
compliance artifacts produced during onboarding. Authenticate first with `request-token`
(see the transactions skill). Base URL: `https://api.cable.tech`.

## Steps

1. **Ingest the entity** — `add-person-info` (`POST /v2/person`) for individuals, or
   `add-company-info` (`POST /v2/company`) for businesses. These carry the `person_id` /
   `company_id` other resources reference.
2. **Ingest identity verification** — `add-identity-verification`
   (`POST /v2/identity_verification`), linked via `related_person_id`.
3. **Ingest the onboarding flow** — `add-onboarding-flow` (`POST /v2/onboarding_flow`), linked to
   the person/company, capturing outcome and reason.
4. **Ingest screening results** — `add-screening` (`POST /v2/screening`) for sanctions/PEP/adverse
   media screening tied to `related_person_id` / `related_company_id`.
5. **Ingest risk assessment** — `add-risk-assessment` (`POST /v2/risk_assessment`) recording the
   financial-crime risk rating for the entity.

## Rules

- **Entities first:** create the person/company before referencing it, or a reference will 404.
- **Auth / errors / idempotency:** bearer token; duplicate id -> 409 (use the `PUT` update
  variants like `update-persons`); custom `GeneralError` envelope; 429 back-off. To delete under
  a data-subject request use `delete-person` and poll `get-deletion-request`.
- See `data-model/cable-data-model.yml`, `conventions/cable-conventions.yml`.
