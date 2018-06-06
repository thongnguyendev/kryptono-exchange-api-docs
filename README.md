# Public Rest API for Kryptono Exchange (2018-06-06)
# General API Information
* The base endpoint is: **https://kryptono.exchange/k/api/**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `404` return codes is used for data not found.
* HTTP `400` return codes is used for invalid data.
* HTTP `406` return codes is used for not acceptable data.
* HTTP `500` return codes is used for invalid format request or wrong from server's side.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "error": -1212,
  "description": "Invalid param."
}
```

* For `GET` endpoints, no parameter is required.
* For `POST`, and `DELETE` endpoints, the parameters must be sent as a `request body` with content type
  `application/json`.
* Parameters may be sent in any order.


# Public API Endpoints
## ENUM definitions
**Order status:**

* OPEN
* FILLED
* PARTIAL_FILL
* CANCELED
* CANCELING

**Order types:**

* LIMIT
* MARKET (currently not supported)
* STOP_LOSS
* TAKE_PROFIT

**Order side:**

* BUY
* SELL

## Header requirements
Name | Value
------------ | ------------
Content-Type | application/json
------------ | ------------
X-Requested-With | XMLHttpRequest
------------ | ------------
Authorization | Your_api_key

## General endpoints
### Test connectivity
```
GET /v1/ping
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Create order
```
POST /v1/order/add_order
```
**Request Body:**
```javascript

```
