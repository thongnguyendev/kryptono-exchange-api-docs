# Public Rest API for Kryptono Exchange (July 4, 2018)
## Update History
### July 4, 2018
* Update API v2

### June 21, 2018
* Update Get Order History apis: [Get Open Orders](#get-open-orders) and [Get Order History](#get-order-history)
* Add Get Order Detail api [Get Order Detail](#get-order-detail)

### June 18, 2018
* Support Market type for create order
* Provide api/websocket for market information, include order books and transaction history: [Get Market Information](#get-market-information)


# General API Information
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in milliseconds.
* The limit for number of requests per minute is 1000.
* HTTP `400` return codes is used for invalid data.
* HTTP `401` return codes is used for unauthorized request.
* HTTP `404` return codes is used for data not found.
* HTTP `406` return codes is used for not acceptable data.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `500` return codes is used for invalid format request or wrong from server's side.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "error": "400xxx",
  "error_description": "Invalid param."
}
```

* For `GET` endpoints, the parameters may be sent as a `request param` or not.
* For `POST`, and `DELETE` endpoints, the parameters must be sent as a `request body` with content type
  `application/json`.
* Parameters may be sent in any order.


# LIMITS
* The `/api/v2/exchange-info` `rate_limits` array contains objects related to the exchange's `REQUESTS` rate limits.
* A `429` will be returned when either rather limit is violated.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a `429` is recieved, it's your obligation as an API to back off and not spam the API.


# Endpoint security type
* Each endpoint has a security type that determines the how you will interact with it.
* API-keys are passed into the Rest API via the `Authorization` header.
* Signature are passed into the Rest API via the `Signature` header.
* API-keys and secret-keys are **case sensitive.**
* API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
MARKET_DATA | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.

* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.


# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `Signature`, to be
  sent in the header.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` or `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specific the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**Tt recommended to use a small recvWindow of 5000 or less!**


## SIGNED Endpoint Examples
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9
secretKey | R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw=

## Example 1: As a query string
* `GET` `/api/v2/account/details`

Parameter | Value
------------ | ------------
timestamp | 1530532714999
recvWindow | 5000

* **Query string:**
  ```
  timestamp=1530532714999&recvWindow=5000
  ```
* **HMAC SHA256 signature:**
  ```
  [linux]$ echo -n 'timestamp=1530532714999&recvWindow=5000' | openssl dgst -sha256 -hmac 'R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw='
  (stdin)= dda4cb640ddc8ff870058b30b5b9faf618f5e465066fcaa906f0a533afc17106
  ```
* **curl command:**
  ```
  (HMAC SHA256)
  [linux]$ curl -X GET 'https://p.kryptono.exchange/k/api/v2/account/details?timestamp=1530532714999&recvWindow=5000' -H 'Authorization: WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9' -H 'Signature: dda4cb640ddc8ff870058b30b5b9faf618f5e465066fcaa906f0a533afc17106' -H 'X-Requested-With: XMLHttpRequest'
  ```

## Example 2: As a request body
* `POST` `/api/v2/order/test`

Parameter | Value
------------ | ------------
order_symbol | KNOW_BTC
order_side | BUY
order_price | 0.00001234
order_size | 1234
type | limit
timestamp | 1530532714999
recvWindow | 5000

* **Request body as json string**
  ```
  {"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}
  ```
* **HMAC SHA256 signature:**
  ```
  [linux]$ echo -n '{"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}' | openssl dgst -sha256 -hmac 'R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw='
  (stdin)= 89c3482ce87fd6cce46a2c4452222a87be1a020f14257da894b55a4366d2a914
  ```
* **curl command:**
  ```
  (HMAC SHA256)
  [linux]$ curl -X POST https://p.kryptono.exchange/k/api/v2/order/test -H 'Authorization: WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9' -H 'Signature: 89c3482ce87fd6cce46a2c4452222a87be1a020f14257da894b55a4366d2a914' -H 'Content-Type: application/json' -H 'X-Requested-With: XMLHttpRequest' -d '{"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}'
  ```
  
  
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
* MARKET
* STOP_LOSS
* TAKE_PROFIT

**Order side:**

* BUY
* SELL

## Header requirements
Name | Value | Note
--------------- | ---------------
Content-Type | application/json | 
X-Requested-With | XMLHttpRequest | 
Authorization | Your_api_key | 
Signature | signature | Required for SIGNED Endpoints

## General endpoints
The base endpoint is: https://p.kryptono.exchange/k/

### Test connectivity
```
GET /api/v2/ping
```
Test connectivity to the Rest API.

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "result": true
}
```

### Check server time
```
GET /api/v2/time
```
Test connectivity to the Rest API and get the current server time.

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "server_time": 1530682662257
}
```

### Exchange information
```
GET /api/v2/exchange-info
```
Current exchange trading rules and symbol information

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "timezone": "UTC",
    "server_time": 1530683054384,
    "rate_limits": [
        {
            "type": "REQUESTS",
            "interval": "MINUTE",
            "limit": 1000
        }
    ],
    "base_currencies": [
        {
            "currency_code": "KNOW",
            "minimum_total_order": "100"
        }
    ],
    "coins": [
        {
            "currency_code": "USDT",
            "name": "Tether",
            "minimum_order_amount": "1"
        }
    ],
    "symbols": [
        {
            "symbol": "GTO_ETH",
            "amount_limit_decimal": 0,
            "price_limit_decimal": 8
        }
    ]
}
```

### Market price
```
GET /api/v2/market-price
```
Get market price for specific symbol or all symbols

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | Default is all symbols

**Response:**
```javascript
[
    {
        "symbol": "GTO_BTC",
        "price": "0.00002542",
        "updated_time": 1530682938651
    }
]
```

## Market data endpoints
The base endpoint is: https://engines.kryptono.exchange/

### Trade History
```
GET /api/v1/ht
```
Get Trade history for specific symbol

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | 

**If symbol is not sent, an empty response will be returned.**

**Resoonse:**
```
{
    "symbol":"KNOW_BTC",
    "limit":100,
    "history":[
        {
            "id":139638,
            "price":"0.00001723",
            "qty":"81.00000000",
            "isBuyerMaker":false,
            "time":1529262196270
        }
    ],
    "time":1529298130192
}
```

### Order Book
```
GET /api/v1/dp
```
Get order book (depth)

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | 

**If symbol is not sent, an empty response will be returned.**

**Response:**
```
{
    "symbol" : "KNOW_BTC",
    "limit" : 100,
    "asks" : [
        [
            "0.00001850",   // price
            "69.00000000"   // size
        ]
    ],
    "bids" : [
        [
            "0.00001651",       // price
            "11186.00000000"    // size
        ]
    ]
    "time" : 1529298130192
}
```








### Get Account's detail
```
GET /v1/account/details
```

**Request Params:**
NONE

**Response:**
```javascript
{
    "account_id": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
    "email": "email@email.com",
    "phone": null,
    "enable_google_2fa": true,
    "status": "offline",
    "create_at": 1524567654822,
    "nick_name": "Nickname 1",
    "chat_id": "xxxx@kryptono.exchange",
    "chat_password": "xxxxxxxxxxx",
    "banks": [],
    "country": "US",
    "language": "en",
    "kyc_status": null,
    "kyc_level": "level1",
    "last_login_history": {
        "id": {
            "timestamp": 1528199468,
            "machineIdentifier": 8990639,
            "processIdentifier": 20156,
            "counter": 7772354,
            "time": 1528199468000,
            "date": 1528199468000,
            "timeSecond": 1528199468
        },
        "account_id": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
        "nick_name": "Nickname 1",
        "email": "email@email.com",
        "ip_address": "xxx.xxx.xxx.xxx",
        "login_at": 1528199468073,
        "os_name": "Mac OS X",
        "browser_name": "Chrome",
        "country": "Country",
        "city": "City",
        "sentEmail": true
    },
    "commission_status": true,
    "account_kyc": null,
    "kyc_reject_infos": [],
    "allow_order": 1,
    "disable_withdraw": 0,
    "referral_id": "XXXXXX",
    "favorite_pairs": [
        "KNOW_ETH"
    ],
    "chat_server": "wss://xxx.kryptono.exchange:xxxx/ws",
    "exchange_fee": {
        "standard_fee": "0.1",
        "know_fee": "0.05"
    }
}
```

### Get Balance list
```
GET /v1/account/balances
```
**Request Params:**
NONE

**Response:**
```javascript
[
    {
        "currency_code": "BTC",
        "address": "2MxctvXExQofAVqakPfBjKqVipfwTqwyQyF",
        "total": "1000.00275",
        "available": "994.5022",
        "in_order": "5.50055"
    }
]
```

### Get Market Price (Updated every 5 seconds)
```
GET /v1/market_price?symbol=KNOW_BTC
```
**Request Params:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | Default is all of symbols

**Response:**
```javascript
[
    {
        "symbol": "KNOW_BTC",
        "price": "0.00010677",
        "updated_time": 1528347586009
    }
]
```

### Get Exchange Information
```
GET /v1/exchange_info
```
**Request Params:**
NONE

**Response:**
```javascript
{
    "base_currencies": [
        {
            "currency_code": "BTC",
            "minimum_total_order": "0.001"
        }
    ],
    "coins": [
        {
            "currency_code": "KNOW",
            "name": "Know",
            "minimum_order_amount": "1"
        }
    ],
    "symbols": [
        {
            "symbol": "KNOW_BTC",
            "amount_limit_decimal": 0,
            "price_limit_decimal": 8
        }
    ]
}
```

### Get Market Information
#### Transaction History
##### API
```
GET https://engines.kryptono.exchange/api/v1/ht?symbol=<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET https://engines.kryptono.exchange/api/v1/ht?symbol=KNOW_BTC
```
```
{
    "symbol":"KNOW_BTC",
    "limit":100,
    "history":[
        {
            "id":139638,
            "price":"0.00001723",
            "qty":"81.00000000",
            "isBuyerMaker":false,
            "time":1529262196270
        }
    ],
    "time":1529298130192
}
```

##### Websocket
```
GET wss://engines.kryptono.exchange/ws/v1/ht/<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET wss://engines.kryptono.exchange/ws/v1/ht/KNOW_USDT
```
```
{
    "c" : 1529298630675,
    "e" : "history_trade",
    "k" : 146851,
    "m" : false,
    "p" : "0.10400000",
    "q" : "107.00000000",
    "s" : "KNOW_USDT",
    "t" : 1529298630370
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
c | LONG | Event time |
e | STRING | Event type |
k | STRING | ID |
m | BOOLEAN | Is order made by buyer |
p | STRING | Price |
q | STRING | Quantity |
s | STRING | Symbol |
t | STRING | Trade time |

#### Order Book Data
##### API
```
GET https://engines.kryptono.exchange/api/v1/dp?symbol=<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET https://engines.kryptono.exchange/api/v1/dp?symbol=KNOW_BTC
```
```
{
    "symbol" : "KNOW_BTC",
    "limit" : 100,
    "asks" : [
        [
            "0.00001850",   // price
            "69.00000000"   // size
        ]
    ],
    "bids" : [
        [
            "0.00001651",       // price
            "11186.00000000"    // size
        ]
    ]
    "time" : 1529298130192
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
symbol | STRING | Symbol |
limit | INTEGER | Limit length of list |
asks | Array | List of Sell Order |
bids | Array | List of Buy Order |
time | LONG | Time |

##### Websocket
```
GET wss://engines.kryptono.exchange/ws/v1/dp/<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET wss://engines.kryptono.exchange/ws/v1/dp/KNOW_USDT
```
```
{
    "a" : [
        [
            "0.10400000",   // price
            "245.00000000"    // size
        ]
    ],
    "b" : [
        [
            "0.10400000", 
            "245.00000000"
        ]
    ]
    "e" : "depthUpdate"
    "s" : "KNOW_USDT"
    "t" : 1529299716639
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
s | STRING | Symbol |
e | STRING | Event type |
a | Array | List of Sell Order |
b | Array | List of Buy Order |
t | LONG | Time |


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

### Get Order Detail
```
POST /v1/order/list/details
```

**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |

**Request Body:**
```javascript
{
  "order_id" : "0e3f05e0-912c-4957-9322-d1a34ef6e312"
}
```

**Response:**
```javascript
{
    "order_id": "0e3f05e0-912c-4957-9322-d1a34ef6e312",
    "account_id": "14ce3690-4e86-4f69-8412-b9fd88535f8z",
    "order_symbol": "KNOW_BTC",
    "order_side": "SELL",
    "status": "open",
    "createTime": 1429514463266,
    "type": "limit",
    "order_price": "0.00001234",
    "order_size": "1000",
    "executed": "0",
    "stop_price": "0.00000000",
    "avg": "0.00001234",
    "total": "0.1234 BTC"
}
```

### Get Open Orders
```
POST /v1/order/list/open_order
```

**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default and Maximum are 20 |
page | INTEGER | NO | Default and first page are 0 |
symbol | STRING | YES | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "symbol" : "KNOW_BTC"
}
```

**Response:**
```javascript
{
    "total": 5,  // total number of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_BTC",
            "order_side": "BUY",
            "status": "open",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "0",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 BTC"
        }
    ]
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------ 
total | INTEGER | Number of pages, only returned in the first page request, other page request returned -1 |


### Get Order History
```
POST /v1/order/list/order_history
```

**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default and Maximum are 20 |
page | INTEGER | NO | Default and first page are 0 |
symbol | STRING | YES | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "symbol" : "KNOW_BTC"
}
```

**Response:**
```javascript
{
    "total": 10,  // total number of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_BTC",
            "order_side": "BUY",
            "status": "filled",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "7777",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 BTC"
        }
    ]
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------ 
total | INTEGER | Number of pages, only returned in the first page request, other page request returned -1 |


### Get Trade History
```
POST /v1/order/list/trade_history
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
type | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "type" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 100,	// total of pages
    "list": [
        {
            "id": {
                "timestamp": 1527488215,
                "machineIdentifier": 8990639,
                "processIdentifier": 29166,
                "counter": 16093657,
                "time": 1527488215000,
                "date": 1527488215000,
                "timeSecond": 1527488215
            },
            "restingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "incomingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "symbol": "GTO_BTC",
            "restingOrderId": "dc8a092e-ab6e-4856-9b95-93eb1d4732ad",
            "incomingOrderId": "6bc57c6d-4552-4ebb-ae9e-5fe75900b300",
            "incomingSide": "BUY",
            "price": "0.00003402",
            "executedQuantity": "500",
            "remainingQuantity": "400",
            "matchingTime": 1527488214898,
            "resting_fee": "0.59885764 KNOW",
            "incoming_fee": "0.66250000 KNOW",
            "total": "0.01701000 BTC"
        }
    ]
}
```

### Get Order's Trade History
```
POST /v1/order/order_trade_detail
```

**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |

**Request Body:**
```javascript
{
  "order_id" : "dc8a092e-ab6e-4856-9b95-93eb1d4732ad"
}
```

**Response:**
```javascript
{
    "total": -1,
    "list": [
        {
            "id": {
                "timestamp": 1527488215,
                "machineIdentifier": 8990639,
                "processIdentifier": 29166,
                "counter": 16093657,
                "time": 1527488215000,
                "date": 1527488215000,
                "timeSecond": 1527488215
            },
            "restingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "incomingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "symbol": "GTO_BTC",
            "restingOrderId": "dc8a092e-ab6e-4856-9b95-93eb1d4732ad",
            "incomingOrderId": "6bc57c6d-4552-4ebb-ae9e-5fe75900b300",
            "incomingSide": "BUY",
            "price": "0.00003402",
            "executedQuantity": "500",
            "remainingQuantity": "400",
            "matchingTime": 1527488214898,
            "resting_fee": "0.59885764 KNOW",
            "incoming_fee": "0.66250000 KNOW",
            "total": "0.01701000 BTC"
        }
    ]
}
```
