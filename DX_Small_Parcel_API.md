# DX_小包裹 (DX Small Parcel) API

## Overview
- Small-parcel API for label creation, rating, and tracking.
- All requests use HTTPS with JSON payloads.

## Base URL
- `https://api.yourdomain.com/v1/dx_parcel`

## Authentication
- Bearer token: `Authorization: Bearer <token>`.

## Idempotency
- For mutation requests, provide `Idempotency-Key` to safely retry.

## Endpoints

### Create Label
- `POST /labels`
- Purpose: create a shipping label and return tracking details.
- Headers: `Content-Type: application/json`, `Authorization`, `Idempotency-Key` (recommended)
- Request body:
  - `shipper` (object, required): `{ name, address1, city, state, postal_code, country, phone }`
  - `recipient` (object, required): same shape as `shipper`
  - `package` (object, required): `{ weight_lb, length_in, width_in, height_in, packaging_type }`
  - `service_level` (string, required): e.g., `GROUND`, `EXPRESS`
  - `declared_value` (number, optional)
  - `reference` (string, optional)
- Response `201 Created`:
  - `label_id` (string)
  - `tracking_number` (string)
  - `service_level` (string)
  - `rate` (number)
  - `currency` (string, ISO 4217)
  - `label_url` (string, PDF/PNG)
- Errors: `400` invalid payload, `401` unauthorized, `409` duplicate reference, `422` rating failed.

#### Sample
```bash
curl -X POST https://api.yourdomain.com/v1/dx_parcel/labels \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 7c8d9e0f-1a2b-3c4d-5b0a-6f7e8d9c0b1a" \
  -d '{
    "shipper": { "name": "Sender", "address1": "1 Main", "city": "LA", "state": "CA", "postal_code": "90001", "country": "US", "phone": "1234567890" },
    "recipient": { "name": "Receiver", "address1": "9 Park", "city": "Dallas", "state": "TX", "postal_code": "75201", "country": "US", "phone": "2140000000" },
    "package": { "weight_lb": 5.2, "length_in": 10, "width_in": 8, "height_in": 6, "packaging_type": "CUSTOM" },
    "service_level": "GROUND",
    "declared_value": 100
  }'
```

### Get Rate
- `POST /rates`
- Purpose: return a price quote without creating a label.
- Headers: `Content-Type: application/json`, `Authorization`
- Request body: same as Create Label (omit `reference`).
- Response `200 OK`:
  - `rate_id` (string)
  - `total_charge` (number)
  - `surcharges` (array): `{ code, amount, description }`
  - `estimated_delivery_date` (string, ISO 8601)
- Errors: `400` invalid payload, `401` unauthorized, `422` no service.

### Track Parcel
- `GET /labels/{label_id}/tracking`
- Purpose: retrieve parcel status and event history.
- Headers: `Authorization`
- Path params: `label_id` (string)
- Response `200 OK`:
  - `label_id` (string)
  - `tracking_number` (string)
  - `status` (string; e.g., `IN_TRANSIT`, `DELIVERED`, `EXCEPTION`)
  - `events` (array): `{ code, description, location, timestamp }`
- Errors: `401` unauthorized, `404` not found.

## HTTP Status Reference
- `200/201`: success
- `400`: validation error
- `401`: auth failed or missing token
- `404`: resource not found
- `409`: duplicate (idempotent key or reference clash)
- `422`: business rule failure (e.g., rating unavailable)

## Rate Limiting
- Default: 100 requests per minute per token.
- Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After` on 429.

## Changelog
- v1.0: initial public specification.

