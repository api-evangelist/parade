---
name: Post available capacity and check carrier onboarding
description: Post a carrier's available truck to Parade for matching and check a carrier's onboarding status with a broker.
api: openapi/parade-digital-transactions-openapi.yaml
operations: [get_carrier, get_carrier_by_mc, add_truck]
---

# Post available capacity and check carrier onboarding (Parade syndication)

Use this to advertise a carrier's empty truck to a broker's loads and to verify the carrier can transact.

## Auth
All calls require the `Authorization` header with the partner token Parade issued you. See
`authentication/parade-authentication.yml`.

## Steps
1. **Check onboarding** — call `get_carrier` with the carrier `dot_number` (or `get_carrier_by_mc` with
   the `mc_number`) and the broker's `company_external_key`. A disqualified/not-onboarded carrier cannot
   quote (`add_quote` will return `406`), so resolve onboarding first.
2. **Post the truck** — call `add_truck` with the `PostedTruck` payload including `broker_external_key`.
   Parade matches the truck to that broker's loads and surfaces it in the broker portal; the carrier may
   also receive matched loads by email if the broker enabled it.

## Rules
- `broker_external_key` / `company_external_key` must be the key Parade issued for that broker.
- Errors follow `errors/parade-problem-types.yml` (`401` bad token, `404` broker not found, `500`/`504` retry).
