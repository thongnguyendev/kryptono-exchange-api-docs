### Account deposit history (USER_DATA)
```
GET /api/v2/account/deposits       (HMAC SHA256)
```
Get deposit history

**Weight:** 1

**Parameters (as query string):**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
currency | STRING | NO |
fromId | STRING | NO | Use the last id from previous repsonse to get more transaction |
limit | LONG | NO | Default and maximum is 50 |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

**Response:**
```javascript
[
    {
        "id": "5afe982c892faf02836008fb",
        "transaction_id": "0x22bcad8167444b1042261d3d2df9f06942dfb8b9c9282e50cb417afea416c8bc",
        "wallet_id": "5ed7ef50-2af2-49bf-8414-8cce5f593c30",
        "currency_code": "KNOW",
        "amount": "38",
        "status": "success",
        "information": {
            "from_address" : "0xa107483c8a16a58871182a48d4ba1fbbb6a64c71",
            "to_address" : "0xce8d51f9ee1b2a4d0cf95972947dd3dc454d6869",
            "txid" : "0x22bcad8167444b1042261d3d2df9f06942dfb8b9c9282e50cb417afea416c8bc"
        },
        "type": "deposit",
        "create_at": "1526634540650",
        "number_confirmations": null,   // ignore
        "total_confirmations": null,    // ignore
        "link_explorer": "https://ropsten.etherscan.io/tx/0x22bcad8167444b1042261d3d2df9f06942dfb8b9c9282e50cb417afea416c8bc"
    }
]
```
