# Spot

## Terminology

* `base asset` refers to the asset that is the `quantity` of a symbol.
* `quote asset` refers to the asset that is the `price` of a symbol.

## ENUM definitions

**Symbol status:**

* TRADING
* HALT
* BREAK

**Symbol type:**

* SPOT

**Asset type:**

* CASH
* MARGIN

**Order status:**

* NEW
* PARTIALLY_FILLED
* FILLED
* CANCELED
* PENDING_CANCEL
* REJECTED

**Order types:**

* LIMIT
* MARKET
* LIMIT_MAKER
* STOP_LOSS (unavailable now)
* STOP_LOSS_LIMIT (unavailable now)
* TAKE_PROFIT (unavailable now)
* TAKE_PROFIT_LIMIT (unavailable now)
* MARKET_OF_PAYOUT (unavailable now)

**Order side:**

* BUY
* SELL

**Time in force:**

* GTC
* IOC
* FOK

**Kline/Candlestick chart intervals:**

  m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**Rate limiters (rateLimitType)**

* REQUESTS_WEIGHT
* ORDERS

**Rate limit intervals**

* SECOND
* MINUTE
* DAY

## General API

### Test connectivity

```shell
GET /openapi/v1/ping
```

Test connectivity to the Rest API.

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{}
```

### Check server time

```shell
GET /openapi/v1/time
```

Test connectivity to the Rest API and get the current server time.

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{
  "serverTime": 1538323200000
}
```

### Broker information

```shell
GET /openapi/v1/brokerInfo
```

Current broker trading rules and symbol information

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{
  "timezone": "UTC",
  "serverTime": 1538323200000,
  "rateLimits": [{
      "rateLimitType": "REQUESTS_WEIGHT",
      "interval": "MINUTE",
      "limit": 1500
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "limit": 20
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "limit": 350000
    }
  ],
  "brokerFilters":[],
  "symbols": [{
    "symbol": "ETHBTC",
    "status": "TRADING",
    "baseAsset": "ETH",
    "baseAssetPrecision": "0.001",
    "quoteAsset": "BTC",
    "quotePrecision": "0.01",
    "icebergAllowed": false,
    "filters": [{
      "filterType": "PRICE_FILTER",
      "minPrice": "0.00000100",
      "maxPrice": "100000.00000000",
      "tickSize": "0.00000100"
    }, {
      "filterType": "LOT_SIZE",
      "minQty": "0.00100000",
      "maxQty": "100000.00000000",
      "stepSize": "0.00100000"
    }, {
      "filterType": "MIN_NOTIONAL",
      "minNotional": "0.00100000"
    }]
  }]
}
```

## Market Data API

### Order book

```shell
GET /openapi/quote/v1/depth
```

**Weight:**
Adjusted based on the limit:

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 100; max 100.

**Caution:** setting limit=0 can return a lot of data.

**Response:**

[PRICE, QTY]

```javascript
{
  "bids": [
    [
      "3.90000000",   // PRICE
      "431.00000000"  // QTY
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "asks": [
    [
      "4.00000200",  // PRICE
      "12.00000000"  // QTY
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

### Recent trades list

```shell
GET /openapi/quote/v1/trades
```

Get recent trades (up to last 500).

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**Response:**

```javascript
[
  {
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true
  }
]
```

### Kline/Candlestick data

```shell
GET /openapi/quote/v1/klines
```

Kline/candlestick bars for a symbol.
Klines are uniquely identified by their open time.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.

**Response:**

```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ]
]
```

### 24hr ticker price change statistics

```shell
GET /openapi/quote/v1/ticker/24hr
```

24 hour price change statistics. **Careful** when accessing this with no symbol.

**Weight:**
1 for a single symbol; **40** when the symbol parameter is omitted

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, tickers for all symbols will be returned in an array.

**Response:**

```javascript
{
  "time": 1538725500422,
  "symbol": "ETHBTC",
  "bestBidPrice": "4.00000200",
  "bestAskPrice": "4.00000200",
  "lastPrice": "4.00000200",
  "openPrice": "99.00000000",
  "highPrice": "100.00000000",
  "lowPrice": "0.10000000",
  "volume": "8913.30000000"
}
```

OR

```javascript
[
  {
    "time": 1538725500422,
    "symbol": "ETHBTC",
    "lastPrice": "4.00000200",
    "openPrice": "99.00000000",
    "highPrice": "100.00000000",
    "lowPrice": "0.10000000",
    "volume": "8913.30000000"
 }
]
```

### Symbol price ticker

```shell
GET /openapi/quote/v1/ticker/price
```

Latest price for a symbol or symbols.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.

**Response:**

```javascript
{
  "price": "4.00000200"
}
```

OR

```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```

### Symbol order book ticker

```shell
GET /openapi/quote/v1/ticker/bookTicker
```

Best price/qty on the order book for a symbol or symbols.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, bookTickers for all symbols will be returned in an array.

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",
  "bidQty": "431.00000000",
  "askPrice": "4.00000200",
  "askQty": "9.00000000"
}
```

OR

```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```

## Account API

### New order  (TRADE)

```shell
POST /openapi/v1/order  (HMAC SHA256)
```

Send in a new order.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
assetType | STRING | NO |
side | ENUM | YES |
type | ENUM | YES |
timeInForce | ENUM | NO |
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | A unique id for the order. Automatically generated if not sent.
stopPrice | DECIMAL | NO | Used with `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders. Unavailable
icebergQty | DECIMAL | NO | Used with `LIMIT`, `STOP_LOSS_LIMIT`, and `TAKE_PROFIT_LIMIT` to create an iceberg order. Unavailable
recvWindow | LONG | NO |
timestamp | LONG | YES |

Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`
`MARKET` | `quantity`
`STOP_LOSS` | `quantity`, `stopPrice`
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice`
`TAKE_PROFIT` | `quantity`, `stopPrice`
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice`
`LIMIT_MAKER` | `quantity`, `price`

Other info:

* `LIMIT_MAKER` are `LIMIT` orders that will be rejected if they would immediately match and trade as a taker.
* `STOP_LOSS` and `TAKE_PROFIT` will execute a `MARKET` order when the `stopPrice` is reached.
* Any `LIMIT` or `LIMIT_MAKER` type order can be made an iceberg order by sending an `icebergQty`.
* Any order with an `icebergQty` MUST have `timeInForce` set to `GTC`.

Trigger order price rules against market price for both MARKET and LIMIT versions:

* Price above market price: `STOP_LOSS` `BUY`, `TAKE_PROFIT` `SELL`
* Price below market price: `STOP_LOSS` `SELL`, `TAKE_PROFIT` `BUY`

**Response:**

```javascript
{
  "orderId": 28,
  "clientOrderId": "6k9M212T12092"
}
```

### Test new order (TRADE)

```shell
POST /openapi/v1/order/test (HMAC SHA256)
```

Test new order creation and signature/recvWindow long.
Creates and validates a new order but does not send it into the matching engine.

**Weight:**
1

**Parameters:**

Same as `POST /openapi/v1/order`

**Response:**

```javascript
{}
```

### Query order (USER_DATA)

```shell
GET /openapi/v1/order (HMAC SHA256)
```

Check an order's status.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

Notes:

* Either `orderId` or `origClientOrderId` must be sent.
* For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "cummulativeQuoteQty": "0.0",
  "avgPrice": "0.0",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.0",
  "icebergQty": "0.0",
  "time": 1499827319559,
  "updateTime": 1499827319559,
  "isWorking": true
}
```

### Cancel order (TRADE)

```shell
DELETE /openapi/v1/order  (HMAC SHA256)
```

Cancel an active order.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
clientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

Either `orderId` or `clientOrderId` must be sent.

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "clientOrderId": "tU721112KM",
  "orderId": 1,
  "status": "CANCELED"
}
```

### Current open orders (USER_DATA)

```shell
GET /openapi/v1/openOrders  (HMAC SHA256)
```

GET all open orders on a symbol. **Careful** when accessing this with no symbol.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* If `orderId` is set, it will get orders < that `orderId`. Otherwise most recent orders are returned.

**Response:**

```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "t7921223K12",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "avgPrice": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### History orders (USER_DATA)

```shell
GET /openapi/v1/historyOrders (HMAC SHA256)
```

GET all orders of the account;  canceled, filled or rejected.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* If `orderId` is set, it will get orders < that `orderId`. Otherwise most recent orders are returned.

**Response:**

```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "987yjj2Ym",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "avgPrice": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### Account information (USER_DATA)

```shell
GET /openapi/v1/account (HMAC SHA256)
```

GET current account information.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{
  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "updateTime": 123456789,
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
```

### Account trade list (USER_DATA)

```shell
GET /openapi/v1/myTrades  (HMAC SHA256)
```

GET trades for a specific account.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from.
toId | LONG | NO | TradeId to fetch to.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* If only `fromId` is set，it will get orders < that `fromId` in descending order.
* If only `toId` is set, it will get orders > that `toId` in ascending order.
* If `fromId` is set and `toId` is set, it will get orders < that `fromId` and > that `toId` in descending order.
* If `fromId` is not set and `toId` it not set, most recent order are returned in descending order.

**Response:**

```javascript
[
  {
    "symbol": "ETHBTC",
    "id": 28457,
    "orderId": 100234,
    "matchOrderId": 109834,
    "price": "4.00000100",
    "qty": "12.00000000",
    "commission": "10.10000000",
    "commissionAsset": "ETH",
    "time": 1499865549590,
    "isBuyer": true,
    "isMaker": false
  }
]
```

### Account deposit list (USER_DATA)

```shell
GET /openapi/v1/depositOrders  (HMAC SHA256)
```

GET deposit orders for a specific account.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | Deposit OrderId to fetch from. Default gets most recent deposit orders.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* If `fromId` is set, it will get orders > that `fromId`. Otherwise most recent orders are returned.

**Response:**

```javascript
[
  {
	"orderId": 100234,
	"token": "EOS",
	"address": "deposit2jb",
	"addressTag": "19012584",
	"fromAddress": "clarkkent",
	"fromAddressTag": "19029901",
	"time": 1499865549590,
	"quantity": "1.01"
  }
]
```

### Sub-account list(SUB_ACCOUNT_LIST)

```shell
POST /openapi/v1/subAccount/query
```

Query sub-account lists

**Parameters:**

None

**Weight:**
5

**Response:**

```javascript
[
    {
        "accountId": "122216245228131",
        "accountName": "",
        "accountType": 1,
        "accountIndex": 0 // main-account: 0, sub-account: 1
    },
    {
        "accountId": "482694560475091200",
        "accountName": "createSubAccountByCurl", // sub-account name
        "accountType": 1, // sub-account type 1. token trading 3. contract trading
        "accountIndex": 1
    },
    {
        "accountId": "422446415267060992",
        "accountName": "",
        "accountType": 3,
        "accountIndex": 0
    },
    {
        "accountId": "482711469199298816",
        "accountName": "createSubAccountByCurl",
        "accountType": 3,
        "accountIndex": 1
    },
]
```

### Internal Account Transfer (ACCOUNT_TRANSFER)

```shell
POST /openapi/v1/transfer
```

Internal transfer

**Weight:**
1

**Parameters:**

Name | Typee | Mandatory | Description
------------ | ------------ | ------------ | ------------
fromAccountType | int | YES |source account type: 1. token trading account 2.Options account 3. Contracts account
fromAccountIndex | int | YES |sub-account index(valid when using main-account api, get sub-account indices from `SUB_ACCOUNT_LIST` endpoint)
toAccountType | int | YES | Target account type: 1. token trading account 2.Options account 3. Contracts account
toAccountIndex | int | YES | sub-account index(valid when using main-account api, get sub-account indices from `SUB_ACCOUNT_LIST` endpoint)
tokenId | STRING | YES | tokenID
amount | STRING | YES | Transfer amount

**Response:**

```javascript
{
    "success":"true" // success
}
```

**Explanation**

1. Either transferring or receiving account must be the main account (Token trading account)

2. Main account api can support transferring to other account(including sub-accounts) and receiving from other accounts

3. **Sub-account API only supports transferring from current account to the main-account. Therefore `fromAccountType\fromAccountIndex\toAccountType\toAccountIndex` should be left empty.**


### Check Balance Flow (BALANCE_FLOW)

```shell
POST /openapi/v1/balance_flow
```

Check blance flow

**Weight:**
5

**Parameters:**

|Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
 | accountType   | int     | no | Account account_type | 默认1 |
 | accountIndex  | int     | no | Account account_index | 默认0 |
 | tokenId       | string  | no     | token_id     | eg: BTC                                |
 | fromFlowId  | long    | no     |  | Query data that id < fromFlowId|
 | endFlowId   | long    | no     |  | Query data that id > |endFlowId  |
| startTime     | long    | no    | Start Time     | Timestamp (millisecond)                             |
| endTime       | long    | no     | End Time     | Timestamp (millisecond)                             |
| limit          | integer | no    | Number of entries   | Default is 50，max is 100                                       |

**Response:**

```javascript
[
    {
        "id": "539870570957903104",
        "accountId": "122216245228131",
        "tokenId": "BTC",
        "tokenName": "BTC",
        "flowTypeValue": 51, // balance flow type
        "flowType": "USER_ACCOUNT_TRANSFER", // balance flow type name
        "flowName": "Transfer", // balance flow type Explanation
        "change": "-12.5", // change
        "total": "379.624059937852365", // total asset after change
        "created": "1579093587214"
    },
    {
        "id": "536072393645448960",
        "accountId": "122216245228131",
        "tokenId": "USDT",
        "tokenName": "USDT",
        "flowTypeValue": 7,
        "flowType": "AIRDROP",
        "flowName": "Airdrop",
        "change": "-2000",
        "total": "918662.0917630848",
        "created": "1578640809195"
    }
]
```

**Explanation**

1. Main-account API can query balance flow for token account and other accounts(including sub-accounts, or designated `accountType` and `accountIndex` accounts)

2. Sub-account API can only query current sub-account, therefore `accountType` and `accountIndex` is not required.

**Please see the following for balance flow types**

Category|Parameter Type Name|Parameter Type Id|Explanation|
------|------|------|------|
General Balance Flow|TRADE|1|trades|
General Balance Flow|FEE|2|trading fees|
General Balance Flow|TRANSFER|3|transfer|
General Balance Flow|DEPOSIT|4|deposit|
Derivatives|MAKER_REWARD|27|maker reward
Derivatives|PNL|28|PnL from contracts
Derivatives|SETTLEMENT|30|Settlement
Derivatives|LIQUIDATION|31|Liquidation
Derivatives|FUNDING_SETTLEMENT|32|期货等的资金费率结算
Internal Transfer|USER_ACCOUNT_TRANSFER|51|userAccountTransfer 专用，流水没有subjectExtId
OTC|OTC_BUY_COIN|65|OTC buy coin
OTC|OTC_SELL_COIN|66|OTC sell coin
OTC|OTC_FEE|73|OTC fees
OTC|OTC_TRADE|200|Old OTC balance flow
Campaign|ACTIVITY_AWARD|67|Campaign reward
Campaign|INVITATION_REFERRAL_BONUS|68|邀请返佣
Campaign|REGISTER_BONUS|69|Registration reward
Campaign|AIRDROP|70|Airdrop
Campaign|MINE_REWARD|71|Mining reward

## User data stream API

Specifics on how user data streams work is in another document.

### Start user data stream (USER_STREAM)

```shell
POST /openapi/v1/userDataStream
```

Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### Keepalive user data stream (USER_STREAM)

```shell
PUT /openapi/v1/userDataStream
```

Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```

### Close user data stream (USER_STREAM)

```shell
DELETE /openapi/v1/userDataStream
```

Close out a user data stream.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```

## Filters

Filters define trading rules on a symbol or an broker.
Filters come in two forms: `symbol filters` and `broker filters`.

### Symbol filters

#### PRICE_FILTER

The `PRICE_FILTER` defines the `price` rules for a symbol. There are 3 parts:

* `minPrice` defines the minimum `price`/`stopPrice` allowed.
* `maxPrice` defines the maximum `price`/`stopPrice` allowed.
* `tickSize` defines the intervals that a `price`/`stopPrice` can be increased/decreased by.

In order to pass the `price filter`, the following must be true for `price`/`stopPrice`:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0

**/brokerInfo format:**

```javascript
  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
```

#### LOT_SIZE

The `LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity`/`icebergQty` allowed.
* `maxQty` defines the maximum `quantity`/`icebergQty` allowed.
* `stepSize` defines the intervals that a `quantity`/`icebergQty` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`/`icebergQty`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0

**/brokerInfo format:**

```javascript
  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

#### MIN_NOTIONAL

The `MIN_NOTIONAL` filter defines the minimum notional value allowed for an order on a symbol.
An order's notional value is the `price` * `quantity`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000"
  }
```

#### MAX_NUM_ORDERS

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a symbol.
Note that both "algo" orders and normal orders are counted for this filter.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ORDERS",
    "limit": 25
  }
```

#### MAX_NUM_ALGO_ORDERS

The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on a symbol.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
```

#### ICEBERG_PARTS

The `ICEBERG_PARTS` filter defines the maximum parts an iceberg order can have. The number of `ICEBERG_PARTS` is defined as `CEIL(qty / icebergQty)`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
```

### Broker Filters

#### BROKER_MAX_NUM_ORDERS

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the broker.
Note that both "algo" orders and normal orders are counted for this filter.

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ORDERS",
    "limit": 1000
  }
```

#### BROKER_MAX_NUM_ALGO_ORDERS

The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on the broker.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ALGO_ORDERS",
    "limit": 200
  }
```