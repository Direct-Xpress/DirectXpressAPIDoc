# DX LTL API (English)

**Document version:** v1.3  
**Protocol:** HTTPS  
**Content type:** JSON (`application/json`)  
**Language:** `X-App-Language: zh | en | es`  
**Auth:** Bearer Token (`Authorization: Bearer <access_token>`)

---

## Table of Contents <a name="index"></a>
- [1. API Standards](#api-standards)
  - [1.1 Response Envelope](#response-envelope)
  - [1.2 HTTP Status Codes](#http-status-codes)
  - [1.3 Optional Signature/Encryption](#signature-encryption)
- [2. API Reference](#api-reference)
  - [2.1 Get Access Token](#get-access-token)
  - [2.2 Refresh Access Token](#refresh-access-token)
  - [2.3 Create Logistics Order](#create-logistics-order)
  - [2.4 Upload Delivery Note](#upload-delivery-note)
- [Changelog](#changelog)

---

## 1. API Standards <a name="api-standards"></a>

### 1.1 Response Envelope <a name="response-envelope"></a>
All endpoints return a consistent JSON envelope:

```json
{
  "code": 0,
  "message": "Success",
  "data": {}
}
```

Field | Type | Required | Description
--- | --- | --- | ---
`code` | integer | Yes | Business code (`0` typically means success)
`message` | string | No | Human-readable message
`data` | object | No | Payload object (varies by endpoint)

### 1.2 HTTP Status Codes <a name="http-status-codes"></a>
HTTP status codes follow standard semantics:

Status | Meaning
--- | ---
200 | Success
4xx | Client error (validation/auth)
5xx | Server error

### 1.3 Optional Signature/Encryption <a name="signature-encryption"></a>
If required by the server, request payloads may be encrypted (e.g., AES + Base64Url) and a signature value passed as a request parameter (e.g., `sign=...`). By default, these APIs use plain JSON over HTTPS.

---

## 2. API Reference <a name="api-reference"></a>

### Common Headers
Header | Required | Notes
--- | --- | ---
`X-App-Language` | Yes | Response language: `zh`, `en`, `es`
`Authorization` | Conditional | `Bearer <access_token>` (required for token refresh and logistics APIs)
`Content-Type` | Conditional | Use `application/json` for requests with JSON body (POST)

---

## 2.1 Get Access Token <a name="get-access-token"></a>
Exchange `app_key` and `app_secret` for an access token.

Item | Value
--- | ---
Method | `POST`
URL | `https://api.bz/ltl/v1/auth/get_access_token`
Auth | None (uses app credentials)
Body | JSON

#### Headers
Header | Required | Example
--- | --- | ---
`Content-Type` | Yes | `application/json`
`X-App-Language` | Yes | `en`

#### Request Body
Field | Type | Required | Validation (recommended)
--- | --- | --- | ---
`app_key` | string | Yes | `^[A-Za-z0-9]{1,100}$`
`app_secret` | string | Yes | `^[A-Za-z0-9]{1,200}$`

#### Example Request
```
POST https://api.bz/ltl/v1/auth/get_access_token
Content-Type: application/json
X-App-Language: en

{
  "app_key": "your_app_key",
  "app_secret": "your_app_secret"
}
```

#### Example Response (200)
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 86400
  }
}
```

#### Notes
- Treat `app_key`/`app_secret` as sensitive secrets. Do not leak them in logs or client bundles.

---

## 2.2 Refresh Access Token <a name="refresh-access-token"></a>
Refresh the access token. **No request body is required.**

Item | Value
--- | ---
Method | `GET`
URL | `https://api.bz/ltl/v1/auth/refresh_access_token`
Auth | Bearer Token
Body | None

#### Headers
Header | Required | Example
--- | --- | ---
`X-App-Language` | Yes | `en`
`Authorization` | Yes | `Bearer your_access_token`

#### Example Request
```
GET https://api.bz/ltl/v1/auth/refresh_access_token
X-App-Language: en
Authorization: Bearer your_access_token
```

#### Example Response (200)
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 86172
  }
}
```

#### Errors
- `401`: invalid or expired token
- `404`: endpoint not found

---

## 2.3 Create Logistics Order <a name="create-logistics-order"></a>
Create a logistics order (pickup/delivery contacts + cargo items).

Item | Value
--- | ---
Method | `POST`
URL | `https://api.bz/ltl/v1/logistics/create_order`
Auth | Bearer Token
Body | JSON

#### Headers
Header | Required | Example
--- | --- | ---
`Content-Type` | Yes | `application/json`
`X-App-Language` | Yes | `en`
`Authorization` | Yes | `Bearer your_access_token`

#### Request Body
Field | Type | Required | Notes
--- | --- | --- | ---
`logisticsList` | array | No | Contact list (typically includes pickup and delivery)
`cargoList` | array | Yes | Cargo items list (must include at least 1 item)
`ref` | string | No | Customer reference number

##### `logisticsList[]`
Field | Type | Required | Notes
--- | --- | --- | ---
`contactName` | string | Yes | Contact name
`companyName` | string | Yes | Company name
`email` | string | Yes | Example validation: `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$`
`mobile` | string | Yes | Example US validation: `^1?\\d{10}$`
`addressType` | number | Yes | `1` = pickup, `2` = delivery
`fullAddress` | string | Yes | Full address string (street/city/state/zip)
`actionTime` | number | Yes | UTC timestamp

##### `cargoList[]`
Field | Type | Required | Notes
--- | --- | --- | ---
`cargo_type` | number | Yes | LTL = `1`
`length` | number | Yes | Non-negative
`width` | number | Yes | Non-negative
`height` | number | Yes | Non-negative
`weight` | number | Yes | Non-negative (per pallet/item)
`quantity` | number | Yes | Positive integer (e.g., `^[1-9]\\d*$`)
`description` | string | Yes | Cargo description (free text)

#### Example Request
```
POST https://api.bz/ltl/v1/logistics/create_order
Content-Type: application/json
X-App-Language: en
Authorization: Bearer your_access_token

{
  "logisticsList": [
    {
      "contactName": "DX test",
      "companyName": "The Home Depot",
      "email": "test@o2o.us",
      "mobile": "9493356666",
      "addressType": 1,
      "fullAddress": "6140 Hamner Ave, Mira Loma, CA 91752",
      "actionTime": 1764737842
    },
    {
      "contactName": "Little",
      "companyName": "The Home Depot",
      "email": "test2@o2o.us",
      "mobile": "9493356666",
      "addressType": 2,
      "fullAddress": "14549 Ramona Ave, Chino, CA 91710",
      "actionTime": 1764719842
    }
  ],
  "cargoList": [
    {
      "cargo_type": 1,
      "length": 40,
      "width": 40,
      "height": 80,
      "weight": 200,
      "quantity": 1,
      "description": "furniture"
    }
  ],
  "ref": "your-ref-123"
}
```

#### Example Response (200)
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "orderId": 1100000000000006
  }
}
```

#### Errors
- `400`: missing/invalid parameters
- `401`: unauthorized (check `Authorization`)
- `404`: endpoint not found
- `500`: server error

---

## 2.4 Upload Delivery Note <a name="upload-delivery-note"></a>
Upload a pickup/delivery note (PDF) for an existing order.

Item | Value
--- | ---
Method | `POST`
URL | `https://api.bz/ltl/v1/logistics/upload_delivery_note`
Auth | Bearer Token
Body | JSON

#### Headers
Header | Required | Example
--- | --- | ---
`Content-Type` | Yes | `application/json`
`X-App-Language` | Yes | `en`
`Authorization` | Yes | `Bearer your_access_token`

#### Request Body
Field | Type | Required | Notes
--- | --- | --- | ---
`orderId` | number | Yes | Positive integer
`stream` | string | Yes | PDF file bytes in Base64 (PDF only)

#### Example Request
```
POST https://api.bz/ltl/v1/logistics/upload_delivery_note
Content-Type: application/json
X-App-Language: en
Authorization: Bearer your_access_token

{
  "orderId": 1100000000000006,
  "stream": "BASE64_OF_PDF"
}
```

#### Example Response (200)
```
{
  "code": 0,
  "message": "Success",
  "data": {}
}
```

#### Errors
- `400`: missing/invalid parameters (e.g., invalid Base64 / non-PDF)
- `401`: unauthorized
- `404`: endpoint not found
- `500`: server error

---

## Changelog <a name="changelog"></a>
- v1.3: Add Upload Delivery Note
- v1.2: Add Create Logistics Order
- v1.1: Clarify Refresh Access Token (header-only)
- v1.0: Initial release
