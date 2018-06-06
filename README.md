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
