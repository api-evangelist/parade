---
name: Quote and book a freight load
description: Search Parade-syndicated loads, submit a carrier quote, and book the load on a broker's behalf.
api: openapi/parade-digital-transactions-openapi.yaml
operations: [search_loads, get_load, add_quote, book_now, get_booking]
---

# Quote and book a freight load (Parade syndication)

Use this to move a carrier from finding a load to a confirmed booking.

## Auth
All calls require the `Authorization` header with the partner token Parade issued you
(`tokenAuth`, apiKey in header). See `authentication/parade-authentication.yml`.

## Steps
1. **Find loads** — call `search_loads` (Load Feed API) with filters such as `src_state`,
   `dst_state`, `equipment` (Van/Flatbed/Reefer), and pickup dates. Paginate with `page`/`limit`
   and read the `x-total-count` header. Treat any load no longer returned as unavailable.
2. **Confirm the load** — call `get_load` with the `external_posting_id` and the carrier `dot_number`
   to fetch current details and allowed actions before acting.
3. **Quote** — call `add_quote` with `company_external_key`, `external_posting_id`, carrier identity
   (`carrier_dot_number`, `carrier_name`, `carrier_email`, `carrier_phone_number`) and `quote_amount`.
   Handle `406` (carrier disqualified) and `410` (load gone) per `errors/parade-problem-types.yml`.
4. **Book** — call `book_now` with the booking payload. A `201` means booked synchronously; a `202`
   means the booking is still being processed (broker not API-integrated) — do not retry, instead
   wait for the digital-conversion webhook or poll `get_booking` with the `reservation_id`.

## Rules
- No idempotency key exists; do not blindly retry `add_quote` (each quote re-notifies the broker) or
  `book_now` on timeout — poll `get_booking` to check state first.
- `external_posting_id` is Parade-internal; never display it to carriers.
