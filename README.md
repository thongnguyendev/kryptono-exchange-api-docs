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
  "error_description": "Invalid param."
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
--------------- | ---------------
Content-Type | application/json
X-Requested-With | XMLHttpRequest
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
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_symbol | STRING | YES | |
order_side | STRING | YES | |
order_price | STRING | YES | |
order_size | STRING | YES | |
stop_price | STRING | NO | Required if type is "stop_loss" or "take_profit" |
type | STRING | YES | |

**Request Body:**
```javascript
{
	"order_symbol" : "KNOW_ETH",
	"order_side" : "BUY",
	"order_price" : "0.0000123",
	"order_size" : "7777",
	"stop_price" : "",
	"type" : "limit"
}
```

**Response:**
```javascript
{
    "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
    "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
    "order_symbol": "KNOW_ETH",
    "order_side": "BUY",
    "status": "open",
    "createTime": 1528277973947,
    "type": "limit",
    "order_price": "0.00001230",
    "order_size": "7777",
    "executed": "0",
    "stop_price": "0.00000000",
    "avg": "0.00001230",
    "total": "0.09565710 ETH"
}
```

### Cancel Order
```
DELETE /v1/order/cancel
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |
order_symbol | STRING | YES | |

**Request Body:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
	"order_symbol" : "KNOW_ETH"
}
```

**Response:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
	"order_symbol" : "KNOW_ETH"
}
```

### Cancel All Order
```
DELETE /v1/order/cancel_all
```

**Request Body:**
```javascript
{}
```

**Response:**
```javascript
200 OK
```

### Get Open Order List
```
POST /v1/order/list/open_order
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default is NO LIMIT |
page | INTEGER | NO | Default is 0 |
from_date | LONG | NO | |
to_date | LONG | NO | |
left | STRING | NO | |
right | STRING | NO | |
order_side | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "order_side" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 10,  // total of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_ETH",
            "order_side": "BUY",
            "status": "open",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "0",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 ETH"
        }
    ]
}
```

### Get Order History
```
POST /v1/order/list/order_history
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default is NO LIMIT |
page | INTEGER | NO | Default is 0 |
from_date | LONG | NO | |
to_date | LONG | NO | |
left | STRING | NO | |
right | STRING | NO | |
order_side | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "order_side" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 10,  // total of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_ETH",
            "order_side": "BUY",
            "status": "filled",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "7777",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 ETH"
        }
    ]
}
```
