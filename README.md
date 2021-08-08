# Public Rest API for Hivelly Exhange (2021-08-08)
# General API Information
* The base endpoint is: **https://api.hivelly.com/**
* All endpoints return either a JSON object or array.
* All time and timestamp related fields are in milliseconds since epoch.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side. See JSON error object.
* HTTP `5XX` return codes are used for internal errors; the issue is on the server side.

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters must be sent as a
  `request body` with content type  `application/x-www-form-urlencoded`.
* Parameters may be sent in any order.

# LIMITS
* Use the HTTP headers in order to understand where the application is at for a given rate limit.
* `x-rate-limit-limit:` the rate limit
* `x-rate-limit-remaining:` the number of requests left for the 1 minute window
* A 429 code will be returned when either rate limit is violated.
* When a 429 is received, it's the consumer's obligation to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 403).**

# Endpoint security type

Security Type | Description
------------ | ------------
PUBLIC | Endpoint requires sending a valid API-Key.
PRIVATE | Endpoint requires sending a valid API-Key and signature.

* Each endpoint has a PUBLIC or PRIVATE security type that determines how API consumers interact with it.
* API-keys are always sent and passed into the Rest API via the `X-ETBS-APIKEY` header.
* API-secrets are used in PRIVATE endpoints to generate `HMAC SHA256` signatures. Request payloads are signed and added to the request as the `signature` parameter.
* API-keys and secret-keys **are case sensitive**.


## PRIVATE Endpoint Examples for POST /api/v1/order
Here is a step-by-step example of how to send a valid signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
symbol | LTC_BTC
side | BUY
type | LIMIT
amount | 1
price | 0.1


### Example: As a request body
* **requestBody:** symbol=LTC_BTC&side=BUY&type=LIMIT&amount=1&price=0.1
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTC_BTC&side=BUY&type=LIMIT&amount=1&price=0.1" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-ETBS-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.hivelly.com/api/v1/order' -d 'symbol=LTC_BTC&side=BUY&type=LIMIT&amount=1&price=0.1&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

# Endpoints
<!--ts-->
  * [Public API Endpoints](#public-api-endpoints)
    * [Terminology](#terminology)
    * [ENUM Definitions](#enum-definitions)
    * [General Endpoints](#general-endpoints)
      * [GET /api/v1/ping](#test-connectivity-public)
      * [GET /api/v1/time](#check-server-time-public)
      * [GET /api/v1/markets](#markets-information-public)
      * [GET /api/v1/orderBook](#order-book-public)
      * [GET /api/v1/trades](#recent-trades-list-public)
      * [GET /api/v1/markets](#markets-information-public)
    * [Account Endpoints](#account-endpoints)
      * [GET /api/v1/account](#account-information-public)
      * [GET /api/v1/orders](#user-orders-list-public)
  * [Private API Endpoints](#private-api-endpoints)
    * [POST /api/v1/orders](#new-order-creation-private)
    * [DELETE /api/v1/orders](#cancel-order-private)
    * [POST /api/v1/withdraw](#withdraw-funds-private)
<!--te-->

# Public API Endpoints
## Terminology
* `base asset` refers to the asset that is the `amount` of a symbol.
* `quote asset` refers to the asset that is the `price` of a symbol.

## ENUM definitions
**Market status:**

* ACTIVE
* DISABLED
* STOPPED

**Order status:**

* NEW
* PARTIALLY_FILLED
* FILLED
* CANCELED

**Order types:**

* LIMIT
* MARKET
* STOP
* STOPLIMIT

**Order side:**

* BUY
* SELL

## General endpoints
### Test connectivity (PUBLIC)
```
GET /api/v1/ping
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{}
```
HTTP Code `200`

### Check server time (PUBLIC)
```
GET /api/v1/time
```
Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1628451083052
}
```
HTTP Code `200`

### Markets information (PUBLIC)
```
GET /api/v1/markets
```
Current exchange markets information.

**Parameters:**
NONE

**Response:**
```javascript
{
  "markets": [{
    "symbol": "ETH_BTC",
    "status": "ACTIVE",
    "baseAsset": "ETH",
    "baseAssetPrecision": 8,
    "quoteAsset": "BTC",
    "quotePrecision": 8,
    "lastPrice": 0.022,
    "volumeInQuoteAsset" : 10000,
    "24changeInQuoteAsset": 150,
    "24changeInPercents": 10
  }]
}
```
HTTP Code `200`

### Order book (PUBLIC)
```
GET /api/v1/orderBook
```

**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Example: `ETH_BTC`
limit | INT | NO | Default 100; max 1000.

**Response:**
```javascript
{
  "bids": [
    [
      "4.00000000",     // price
      "431.00000000"   // amount
    ]
  ],
  "asks": [
    [
      "4.00000200",    // price
      "12.00000000"   // amount
    ]
  ]
}
```
HTTP Code `200`

### Recent trades list (PUBLIC)
```
GET /api/v1/trades
```
Get recent trades by market. Data is returned in ascending order. Oldest first, newest last.


**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Example: `ETH_BTC`
limit | INT | NO | Default 100; max 1000. Valid limits:[5, 10, 20, 50, 100, 500, 1000]

**Response:**
```javascript
{
  "trades": [
    {
      "id": 1,
      "side": "BUY",
      "price": "0.10000000",
      "amount": "1.00000000",
      "time": 1543348437000
    }, {
      "id": 2,
      "side": "BUY",
      "price": "0.10000000",
      "amount": "1.00000000",
      "time": 1543348437000
    }, {
      "id": 3,
      "side": "SELL",
      "price": "0.10000000",
      "amount": "1.00000000",
      "time": 1543348437000
    }, {
      "id": 4,
      "side": "BUY",
      "price": "0.10000000",
      "amount": "1.00000000",
      "time": 1543348437000
    }, {
      "id": 5,
      "side": "SELL",
      "price": "0.10000000",
      "amount": "1.00000000",
      "time": 1543348437000
    }
  ]
}
```
HTTP Code `200`

## Account endpoints
### Account information (PUBLIC)
```
GET /api/v1/account
```
Get current account information.

**Parameters:**
NONE

**Response:**
```javascript
{
  "rawUserInfo": {
    "canTrade": true,
    "canWithdraw": true,
    "canDeposit": true,
    "walletsList": [
      {
        "walletId": 1,
        "balancesList": [
          {
            "asset": "BTC",
            "free": "4723846.89208129",
            "locked": "25.00623000"
          },
          {
            "asset": "LTC",
            "free": "4763368.68006011"
            "locked": "25.00623000"
          }
        ]
      }, {
        "walletId": 2,
        "balances": [
          {
            "asset": "BTC",
            "free": "4723846.89208129"
            "locked": "25.00623000"
          },
          {
            "asset": "LTC",
            "free": "4763368.68006011"
            "locked": "25.00623000"
          }
        ]
      }
    ]
  }
}
```
HTTP Code `200`

### User orders list (PUBLIC)
```
GET /api/v1/orders
```
Get recent orders by user wallet ID. Data is returned in ascending order. Oldest first, newest last.


**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
walletId | INT | YES |
status | ENUM | NO | Example: `PARTIALLY_FILLED` return only partially filled orders
type | ENUM | NO | Example: `MARKET` return only market orders
symbol | STRING | NO | Filters orders by market symbol
limit | INT | NO | Min 100, max 1000. Default: 500

**Response:**
```javascript
{
  "userOrders": [
    {
      "symbol": "ETH_BTC",
      "orderId": "deee7d0deb3b987d6db93163d5ce1de3",
      "price": "4.00000100",
      "amount": "12.00000000",
      "usedAmount": "0.00000000",
      "status": "NEW",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": null,
      "time": 1543349010802,
      "updateTime": 1543349011699
    }, {
      "symbol": "ETH_BTC",
      "orderId": "deee7d0deb3b987d6db93163d5ce12e3",
      "price": "4.00000100",
      "amount": "12.00000000",
      "usedAmount": "0.00000000",
      "status": "CANCELED",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": null,
      "time": 1543349010802,
      "updateTime": 1543349011699
    }, {
      "symbol": "ETH_BTC",
      "orderId": "deee7d0deb3b987d6db93143d5ce1de3",
      "price": "4.00000100",
      "amount": "12.00000000",
      "usedAmount": "0.00000000",
      "status": "NEW",
      "type": "MARKET",
      "side": "SELL",
      "stopPrice": null,
      "time": 1543349010802,
      "updateTime": 1543349011699
    }
  ]
}
```
HTTP Code `200`

# Private API Endpoints
### New order creation (PRIVATE)
```
POST /api/v1/orders
```
Create new order

**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
walletId | INT | YES | Used with ALL order types.
symbol | STRING | YES | Used with ALL order types.
side | ENUM | YES | Used with ALL order types.
type | ENUM | YES | Used with ALL order types.
amount | DECIMAL | YES | Used with ALL order types.
price | DECIMAL | NO | Used with `LIMIT` `STOP`, `STOPLIMIT` order types.
stopPrice | DECIMAL | NO | Used with `STOP`, `STOPLIMIT` order types.
signature | STRING | yes | Used with ALL order types.

Additional requirements parameters based on `type`:

Type | Required parameters
------------ | ------------
`LIMIT` | `amount`, `price`
`MARKET` | `amount`
`STOP` | `amount`, `stopPrice`
`STOPLIMIT` | `amount`,  `price`, `stopPrice`

Other info:

* `STOP` will execute a `MARKET` order when the `stopPrice` is reached.
* `STOPLIMIT` will execute a `LIMIT` order when the `stopPrice` is reached.
* `price`, `amount` or `stopPrice` should be more or equals 0.00000001.

**Response:**
```javascript
{
  "newOrder": {
    "orderId": "8c4449129f1c3c372f2d1886ae84fc8d",
    "symbol": "BTC_USDT",
    "price": "1.00000000",
    "originalAmount": "10.00000000",
    "executedAmount": "10.00000000",
    "status": "FILLED",
    "type": "MARKET",
    "side": "SELL",
    "stopPrice": null,
    "transactTime": 1507725176595
  }
}
```
HTTP Code `201`

### Cancel order (PRIVATE)
```
DELETE /api/v1/orders
```
Cancel an active order.

**Parameters:**

Name | Type | Required
------------ | ------------ | ------------
walletId | INT | YES
orderId | STRING | YES
signature | STRING | YES | Used with ALL order types.


**Response:**
```javascript
{}
```
HTTP Code `204`

### Withdraw funds (PRIVATE)
```
POST /api/v1/withdraw
```
Withdraw your funds. ONLY for whitelisted addresses @ **https://www.hivelly.com/settings/withdrawal-addresses**

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
currency | STRING | YES | `BTC`
address | STRING | YES | `3NrRvQ4uss5P1Q2K6qPVUaxC2qaHr6AfYd`
amount | DECIMAL | YES | `0.1`
walletId | INT | YES | `23`
signature | STRING | YES | `c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71`


**Response:**
```javascript
{
  "withdrawalId": "43bc675c7121ebb69171f7dc48fb93de"
}
```
HTTP Code `200`
