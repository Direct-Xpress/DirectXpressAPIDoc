# DX_LTL API

## Overview
- LTL (less-than-truckload) shipping API covering rating, shipment creation, and tracking.
- Use HTTPS for all requests; requests and responses are JSON unless noted.

## Base URL
- `https://api.bz/ltl`

## Authentication
- Bearer token in header: `Authorization: Bearer <token>`.
- Tokens are project-scoped; request revocation if leaked.

## Idempotency
- For mutation endpoints, send `Idempotency-Key` (UUID) to safely retry.

## Endpoints

### Get Access Token
- `POST https://api.bz/ltl/v1/auth/get_access_token`
- Purpose: obtain an access token using application credentials.
- Notes:
  - Keep `app_key` and `app_secret` secure; do not expose in logs or client bundles.
  - Headers must include `Content-Type: application/json` so the server parses the body.
  - `X-App-Language` determines response language (`zh`, `en`, `es`); ensure it is set.
- Headers:
  - `Content-Type` (string, required): `application/json`
  - `X-App-Language` (string, required): target language code (e.g., `en`)
- Request body (JSON):
  - `app_key` (string, required): provided by DX
  - `app_secret` (string, required): secret provided by DX
- Sample request
```bash
curl -X POST https://api.bz/ltl/v1/auth/get_access_token \
  -H "Content-Type: application/json" \
  -H "X-App-Language: en" \
  -d '{
    "app_key": "sFpB619dM3NKI98d8K015ZrwbCPTL2Vs",
    "app_secret": "go8CADu1XGevFMikLAiL3DXFyUrEuh5KjGcqeH2ulj1lpHGJacFxRtpbhUkqP7Vq"
  }'
```
- Success response (`200`)
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjozLCJpZGVudGl0eSI6InJuamhrdnQ4Y2RvdnRvdzAiLCJyb2xlIjoiY3VzdG9tZXIiLCJpc3MiOiJkeHByZXNzIiwiZXhwIjoxNzY0ODI3ODI2LCJuYmYiOjE3NjQ3NDE0MjYsImlhdCI6MTc2NDc0MTQyNn0.9g8CP3UFuOZinBB-CceG4wF0a5Z41lBO-Wd84RqotvU",
    "expires_in": 86400
  }
}
```
- Client-side validation suggestions:
  - `app_key`: validate length 1–100, alphanumeric (e.g., `/^[A-Za-z0-9]{1,100}$/`).
  - `app_secret`: validate length 1–200, alphanumeric (e.g., `/^[A-Za-z0-9]{1,200}$/`).
  - Headers: enforce `Content-Type: application/json` and `X-App-Language` is one of `zh|en|es`.

### Create Shipment
- `POST /shipments`
- Purpose: create an LTL shipment and return identifiers and estimated charges.
- Headers: `Content-Type: application/json`, `Authorization`, `Idempotency-Key` (recommended)
- Request body:
  - `shipper` (object, required): `{ name, address1, city, state, postal_code, country, phone }`
  - `consignee` (object, required): same shape as `shipper`
  - `freight` (array, required): each `{ description, nmfc, class, weight_lb, length_in, width_in, height_in, stackable }`
  - `accessorials` (array<string>, optional): e.g., `["liftgate_pickup","inside_delivery"]`
  - `pickup_date` (string, ISO 8601, required)
  - `reference` (string, optional): customer reference for deduplication
- Response `201 Created`:
  - `shipment_id` (string)
  - `bol_number` (string)
  - `estimated_charge` (number)
  - `currency` (string, ISO 4217)
  - `tracking_url` (string)
- Errors: `400` invalid payload, `401` unauthorized, `409` duplicate reference, `422` rating failed.

#### Sample
```bash
curl -X POST https://api.yourdomain.com/v1/dx_ltl/shipments \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 5b0a1c2e-3d4f-5a6b-7c8d-9e0f1a2b3c4d" \
  -d '{
    "shipper": { "name": "ABC Co", "address1": "1 Main", "city": "LA", "state": "CA", "postal_code": "90001", "country": "US", "phone": "1234567890" },
    "consignee": { "name": "XYZ LLC", "address1": "9 Park", "city": "Dallas", "state": "TX", "postal_code": "75201", "country": "US", "phone": "2140000000" },
    "freight": [{ "description": "Pallets", "nmfc": "12345", "class": "70", "weight_lb": 1200, "length_in": 48, "width_in": 40, "height_in": 60, "stackable": true }],
    "accessorials": ["liftgate_pickup"],
    "pickup_date": "2025-12-17",
    "reference": "PO-123456"
  }'
```

### Get Quote
- `POST /quotes`
- Purpose: price an LTL shipment without creating it.
- Headers: `Content-Type: application/json`, `Authorization`
- Request body: same as Create Shipment (omit `reference`).
- Response `200 OK`:
  - `quote_id` (string)
  - `total_charge` (number)
  - `surcharges` (array): `{ code, amount, description }`
  - `transit_days` (integer)
- Errors: `400` invalid payload, `401` unauthorized, `422` no service.

### Track Shipment
- `GET /shipments/{shipment_id}/tracking`
- Purpose: retrieve current status and history.
- Headers: `Authorization`
- Path params: `shipment_id` (string)
- Response `200 OK`:
  - `shipment_id` (string)
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

