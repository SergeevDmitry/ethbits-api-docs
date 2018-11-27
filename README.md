# Public Rest API for Ethbits (2018-11-15)
# General API Information
* The base endpoint is: **https://api.ethbits.com**
* All endpoints return either a JSON object or array.
* All time and timestamp related fields are in milliseconds.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side. See JSON error object.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  Ethbits's side.


* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `request body` with content type  `application/x-www-form-urlencoded`.
* Parameters may be sent in any order.

# LIMITS
* Use the HTTP headers in order to understand where the application is at for a given rate limit.
* `x-rate-limit-limit:` the rate limit
* `x-rate-limit-remaining:` the number of requests left for the 1 minute window
* A 429 code will be returned when either rate limit is violated.
* When a 429 is recieved, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 403).**

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-ETBS-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.


Security Type | Description
------------ | ------------
PUBLIC | Endpoint requires sending a valid API-Key.
PRIVATE | Endpoint requires sending a valid API-Key and signature.


## PRIVATE Endpoint Examples for POST /api/v1/order
Here is a step-by-step example of how to send a vaild signed payload from the
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
    [linux]$ curl -H "X-ETBS-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.ethbits.com/api/v1/order' -d 'symbol=LTC_BTC&side=BUY&type=LIMIT&amount=1&price=0.1&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

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
  "serverTime": 1499827399559
}
```

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

### Order book (PUBLIC)
```
GET /api/v1/orderBook
```

**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Example: `ETH_BTC`
limit | INT | NO | Default 100; max 1000. Valid limits:[5, 10, 20, 50, 100, 500, 1000]

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
[
  {
    "id": 28457,
    "price": "4.00000100",
    "amount": "12.00000000",
    "time": 1499865549590
  }
]
```

## Account endpoints
### Recent trades list (PUBLIC)
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

**Response:**
```javascript
[
  {
    "id": "8c4449129f1c3c372f2d1886ae84fc8d",
    "status": "NEW",
    "type": "LIMIT",
    "price": "4.00000100",
    "amount": "12.00000000",
    "time": 1499865549590
  }
]
```

### New order creation (PRIVATE)
```
POST /api/v1/order
```
Create new order

**Parameters:**

Name | Type | Required | Description
------------ | ------------ | ------------ | ------------
waleltId | INT | YES | Used with ALL order types.
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

**Response:**
```javascript
{
  "id": "8c4449129f1c3c372f2d1886ae84fc8d",
  "symbol": "BTC_USDT",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "originalAmount": "10.00000000",
  "executedAmount": "10.00000000",
  "status": "FILLED",
  "type": "MARKET",
  "side": "SELL"
}
```

### Cancel order (PRIVATE)
```
DELETE /api/v1/order
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

### Account information (PUBLIC)
```
GET /api/v1/account
```
Get current account information.

**Parameters:**

NO

**Response:**
```javascript
{

  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "wallets": [
    {
      "walletId": 1,
      "balances": [
        {
          "asset": "BTC",
          "free": "4723846.89208129",
          "locked": "0.00000000"
        },
        {
          "asset": "LTC",
          "free": "4763368.68006011",
          "locked": "0.00000000"
        }
      ]
    }, {
      "walletId": 2,
      "balances": [
        {
          "asset": "BTC",
          "free": "4723846.89208129",
          "locked": "0.00000000"
        },
        {
          "asset": "LTC",
          "free": "4763368.68006011",
          "locked": "0.00000000"
        }
      ]
    }
  ]
}
```
