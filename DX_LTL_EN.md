# DX LTL API Reference
**AWS / Stripe Style API Documentation**

**Version:** v1.3  
**Base URL:** https://api.bz  
**Protocol:** HTTPS  
**Format:** JSON  
**Authentication:** Bearer Token  

---

## Overview
DX LTL APIs provide authentication, logistics order creation, and delivery document upload for LTL (Less-Than-Truckload) shipments.

All requests must be made over HTTPS. Responses use a consistent JSON envelope and standard HTTP status codes.

---

## Authentication
DX LTL uses **Bearer Token authentication**, similar to AWS and Stripe APIs.

### Header
```
Authorization: Bearer <access_token>
```

### Token Lifecycle
- Obtain token via **Get Access Token**
- Refresh before expiration using **Refresh Access Token**
- Tokens are scoped per `app_key`

---

## API Conventions

### Request Headers
| Header | Required | Description |
|------|--------|-------------|
| Content-Type | Yes | application/json |
| Authorization | Conditional | Bearer token |
| X-App-Language | Yes | zh / en / es |

---

### Response Format
```json
{
  "code": 0,
  "message": "Success",
  "data": {}
}
```

| Field | Type | Description |
|-----|-----|-------------|
| code | integer | Business status code (`0` = success) |
| message | string | Status message |
| data | object | Response payload |

---

## Authentication APIs

### Get Access Token
```
POST /ltl/v1/auth/get_access_token
```

#### Request Body
```json
{
  "app_key": "your_app_key",
  "app_secret": "your_app_secret"
}
```

#### Response
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 86400
  }
}
```

---

### Refresh Access Token
```
GET /ltl/v1/auth/refresh_access_token
```

#### Headers
```
Authorization: Bearer <access_token>
```

#### Response
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 86172
  }
}
```

---

## Logistics APIs

### Create Logistics Order
```
POST /ltl/v1/logistics/create_order
```

#### Request Body
```json
{
  "logisticsList": [
    {
      "contactName": "DX Test",
      "companyName": "The Home Depot",
      "email": "test@o2o.us",
      "mobile": "9493356666",
      "addressType": 1,
      "fullAddress": "6140 Hamner Ave, Mira Loma, CA 91752",
      "actionTime": 1764737842
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

#### Response
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "orderId": 1100000000000006
  }
}
```

---

### Upload Delivery Note
```
POST /ltl/v1/logistics/upload_delivery_note
```

#### Request Body
```json
{
  "orderId": 1100000000000006,
  "stream": "BASE64_PDF_CONTENT"
}
```

---

## Error Handling

| HTTP Status | Code | Description |
|------------|------|-------------|
| 200 | 0 | Success |
| 400 | -1 | Invalid request |
| 401 | -2 | Unauthorized |
| 404 | -3 | Not found |
| 500 | -9 | Internal server error |

---

## Changelog
- v1.3 Add delivery note upload
- v1.2 Add logistics order API
- v1.1 Clarify refresh token behavior
- v1.0 Initial release
