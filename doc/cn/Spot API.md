# 币币交易

## 术语解释

* `base asset` 指的是symbol的`quantity`（即数量）。

* `quote asset` 指的是symbol的`price`（即价格）。

## ENUM 定义

**Symbol 状态:**

* TRADING - 交易中
* HALT - 终止
* BREAK - 断开

**Symbol 类型:**

* SPOT - 现货

**资产类型:**

* CASH - 现金
* MARGIN - 保证金

**订单状态:**

* NEW - 新订单，暂无成交
* PARTIALLY_FILLED - 部分成交
* FILLED - 完全成交
* CANCELED - 已取消
* PENDING_CANCEL - 等待取消
* REJECTED - 被拒绝

**订单类型:**

* LIMIT - 限价单
* MARKET - 市价单
* LIMIT_MAKER - maker限价单
* STOP_LOSS (unavailable now)  - 暂无
* STOP_LOSS_LIMIT (unavailable now) - 暂无
* TAKE_PROFIT (unavailable now) - 暂无
* TAKE_PROFIT_LIMIT (unavailable now) - 暂无
* MARKET_OF_PAYOUT (unavailable now) - 暂无

**订单方向:**

* BUY - 买单
* SELL - 卖单

**订单时效类型:**

* GTC
* IOC
* FOK

**k线/烛线图区间:**

  m -> 分钟; h -> 小时; d -> 天; w -> 周; M -> 月

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

**频率限制类型 (rateLimitType)**

* REQUESTS_WEIGHT
* ORDERS

**频率限制区间**

* SECOND
* MINUTE
* DAY

## 通用接口

### 测试连接

```shell
GET /openapi/v1/ping
```

测试REST API的连接。

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{}
```

### 服务器时间

```shell
GET /openapi/v1/time
```

测试连接并获取当前服务器的时间。

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

### Broker信息

```shell
GET /openapi/v1/brokerInfo
```

当前broker交易规则和symbol信息

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

## 行情接口

### 订单簿

```shell
GET /openapi/quote/v1/depth
```

**Weight:**

根据limit不同：

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 默认 100; 最大 100.

**注意:** 如果设置limit=0会返回很多数据。

**Response:**

[价格, 数量]

```javascript
{
  "bids": [
    [
      "3.90000000",   // 价格
      "431.00000000"  // 数量
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "asks": [
    [
      "4.00000200",  // 价格
      "12.00000000"  // 数量
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

### 最近成交

```shell
GET /openapi/quote/v1/trades
```

获取当前最新成交（最多500）

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
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

### k线/烛线图数据

```shell
GET /openapi/quote/v1/klines
```

symbol的k线/烛线图数据
K线会根据开盘时间而辨别。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | 默认500; 最大1000.

* 如果startTime和endTime没有发送，只有最新的K线会被返回。

**Response:**

```javascript
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价
    "148976.11427815",  // 交易量
    1499644799999,      // 收盘时间
    "2434.19055334",    // Quote asset数量
    308,                // 交易次数
    "1756.87402397",    // Taker buy base asset数量
    "28.46694368"       // Taker buy quote asset数量
  ]
]
```

### 24小时ticker价格变化数据

```shell
GET /openapi/quote/v1/ticker/24hr
```

24小时价格变化数据。**注意** 如果没有发送symbol，会返回很多数据。

**Weight:**

如果只有一个symbol，1; 如果symbol没有被发送，**40**。

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol没有被发送，所有symbol的数据都会被返回。

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

### Symbol价格

```shell
GET /openapi/quote/v1/ticker/price
```

单个或多个symbol的最新价。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol没有发送，所有symbol的最新价都会被返回。

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

### Symbol最佳订单簿价格

```shell
GET /openapi/quote/v1/ticker/bookTicker
```

单个或者多个symbol的最佳买单卖单价格。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol没有被发送，所有symbol的最佳订单簿价格都会被返回。

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

## 账户接口

### 创建新订单  (TRADE)

```shell
POST /openapi/v1/order  (HMAC SHA256)
```

发送一个新的订单

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
assetType | STRING | NO |
side | ENUM | YES |
type | ENUM | YES |
timeInForce | ENUM | NO |
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 一个自己给订单定义的ID，如果没有发送会自动生成。
stopPrice | DECIMAL | NO | 与 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 和`TAKE_PROFIT_LIMIT` 订单一起使用. **当前不可用**
icebergQty | DECIMAL | NO | 与 `LIMIT`, `STOP_LOSS_LIMIT`, 和 `TAKE_PROFIT_LIMIT` 来创建冰山订单. **当前不可用**
recvWindow | LONG | NO |
timestamp | LONG | YES |

在`type`上的额外强制参数:

类型 | 额外强制参数
------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`
`MARKET` | `quantity`
`STOP_LOSS` | `quantity`, `stopPrice`  **当前不可用**
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice` **当前不可用**
`TAKE_PROFIT` | `quantity`, `stopPrice` **当前不可用**
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **当前不可用**
`LIMIT_MAKER` | `quantity`, `price`


**Response:**

```javascript
{
  "orderId": 28,
  "clientOrderId": "6k9M212T12092"
}
```

### 测试新订单 (TRADE)

```shell
POST /openapi/v1/order/test (HMAC SHA256)
```

用signature和recvWindow测试生成新订单。
创建和验证一个新订单但是不送入撮合引擎。

**Weight:**
1

**Parameters:**

和 `POST /openapi/v1/order`一样。

**Response:**

```javascript
{}
```

### 查询订单 (USER_DATA)

```shell
GET /openapi/v1/order (HMAC SHA256)
```

查询订单状态。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

Notes:

* 单一 `orderId` 或者 `origClientOrderId` 必须被发送。
* 对于某些历史数据 `cummulativeQuoteQty` 可能会 < 0, 这说明数据当前不可用。

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

### 取消订单 (TRADE)

```shell
DELETE /openapi/v1/order  (HMAC SHA256)
```

取消当前正在交易的订单。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
clientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

单一 `orderId` 或者 `clientOrderId`必须被发送。

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "clientOrderId": "tU721112KM",
  "orderId": 1,
  "status": "CANCELED"
}
```

### 当前订单(USER_DATA)

```shell
GET /openapi/v1/openOrders  (HMAC SHA256)
```

获取当前单个或者多个symbol的当前订单。**注意** 如果没有发送symbol，会返回很多数据。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
limit | INT | NO | 默认 500; 最多 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`设定好了，会筛选订单小于`orderId`的。否则会返回最近的订单信息。

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

### 历史订单 (USER_DATA)

```shell
GET /openapi/v1/historyOrders (HMAC SHA256)
```
获取当前账户的所有订单。亦或是取消的，完全成交的，拒绝的。

**Weight:**
5

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`设定好了，会筛选订单小于`orderId`的。否则会返回最近的订单信息。

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

### 账户信息 (USER_DATA)

```shell
GET /openapi/v1/account (HMAC SHA256)
```

获取当前账户信息

**Weight:**
5

**Parameters:**

名称 | 类型 | 是否强制 | 描述
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

### 账户交易记录 (USER_DATA)

```shell
GET /openapi/v1/myTrades  (HMAC SHA256)
```

获取当前账户历史成交记录

**Weight:**
5

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from.
toId | LONG | NO | TradeId to fetch to.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果只有`fromId`，会返回订单号小于`fromId`的，倒序排列。
* 如果只有`toId`，会返回订单号小于`toId`的，升序排列。
* 如果同时有`fromId`和`toId`, 会返回订单号在`fromId`和`toId`的，倒序排列。
* 如果`fromId`和`toId`都没有，会返回最新的成交记录，倒序排列。
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

### 账户存款记录 (USER_DATA)

```shell
GET /openapi/v1/depositOrders  (HMAC SHA256)
```

获取当前账户的存款记录

**Weight:**
5

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | 从哪个OrderId起开始抓取。默认抓取最新的存款记录。
limit | INT | NO | 默认 500; 最大 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`设定好了，会筛选订单小于`orderId`的。否则会返回最近的订单信息。

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
### 子账户列表(SUB_ACCOUNT_LIST)

```shell
POST /openapi/v1/subAccount/query
```

查询子账户列表

**Parameters:**

无

**Weight:**
5

**Response:**

```javascript
[
    {
        "accountId": "122216245228131",
        "accountName": "",
        "accountType": 1,
        "accountIndex": 0 // 账户index 0 默认账户 >0, 创建的子账户
    },
    {
        "accountId": "482694560475091200",
        "accountName": "createSubAccountByCurl", // 子账户名称
        "accountType": 1, // 子账户类型 1 币币账户 3 合约账户
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
### 账户内转账 (ACCOUNT_TRANSFER)

```shell
POST /openapi/v1/transfer
```

转账

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
fromAccountType | int | YES |源账户类型, 1 钱包(币币)账户 2 期权账户 3 合约账户
fromAccountIndex | int | YES |子账户index, 主账户Api调用时候有用，从子账户列表接口获取
toAccountType | int | YES | 目标账户类型, 1 钱包(币币)账户 2 期权账户 3 合约账户
toAccountIndex | int | YES | 子账户index, 主账户Api调用时候有用，从子账户列表接口获取
tokenId | STRING | YES | tokenID
amount | STRING | YES | 转账数量

**Response:**

```javascript
{
    "success":"true" // 0成功
}
```

**说明**

1、转账账户和收款账户的其中一方，必须是主账户(钱包账户)

2、主账户Api可以从钱包账户向其他账户(包括子账户)转账，也可以从其他账户向钱包账户转账

3、**子账户Api调用的时候只能从当前子账户向主账户(钱包账户)转账，所以fromAccountType\fromAccountIndex\toAccountType\toAccountIndex不用填**

### 查询流水 (BALANCE_FLOW)

```shell
POST /openapi/v1/balance_flow
```

查询账户流水

**Weight:**
5

**Parameters:**

|名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
 | accountType   | int     | 否 | 账户对应的account_type | 默认1 |
 | accountIndex  | int     | 否 | 账户对应的account_index | 默认0 |
 | tokenId       | string  | 否     | token_id     | eg: BTC                                |
 | fromFlowId  | long    | 否     | 顺向查询数据 | 指定查询 id < fromFlowId的数据 |
 | endFlowId   | long    | 否     | 反向查询数据 | 指定查询 id > |endFlowId的数据  |
| startTime     | long    | 否     | 开始时间     | 毫秒时间戳                             |
| endTime       | long    | 否     | 结束时间     | 毫秒时间戳                             |
| limit          | integer | 否     | 每页记录数   | 默认50，最大100                                       |

**Response:**

```javascript
[
    {
        "id": "539870570957903104",
        "accountId": "122216245228131",
        "tokenId": "BTC",
        "tokenName": "BTC",
        "flowTypeValue": 51, // 流水类型
        "flowType": "USER_ACCOUNT_TRANSFER", // 流水类型名称
        "flowName": "Transfer", // 流水类型说明
        "change": "-12.5", // 变动值
        "total": "379.624059937852365", // 变动后当前tokenId总资产
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

**说明**

1、主账户Api可以查询钱包账户或者其他账户(包括子账户，指定accountType和accountIndex)的流水’

2、子账户Api只能查询当前子账户的流水，所以不用指定accountType和accountIndex

**流水类型说明请见如下**

归类|类型参数名|类型参数代号|解释说明|
------|------|------|------|
通用流水类|TRADE|1|交易|
通用流水类|FEE|2|交易手续费|
通用流水类|TRANSFER|3|转账|
通用流水类|DEPOSIT|4|充值|
衍生品业务|MAKER_REWARD|27|maker奖励
衍生品业务|PNL|28|期货等的盈亏
衍生品业务|SETTLEMENT|30|交割
衍生品业务|LIQUIDATION|31|强平
衍生品业务|FUNDING_SETTLEMENT|32|期货等的资金费率结算
用户子账户之间内部转账|USER_ACCOUNT_TRANSFER|51|userAccountTransfer 专用，流水没有subjectExtId
OTC|OTC_BUY_COIN|65|OTC 买入coin
OTC|OTC_SELL_COIN|66|OTC 卖出coin
OTC|OTC_FEE|73|OTC 手续费
OTC|OTC_TRADE|200|旧版 OTC 流水
活动|ACTIVITY_AWARD|67|活动奖励
活动|INVITATION_REFERRAL_BONUS|68|邀请返佣
活动|REGISTER_BONUS|69|注册送礼
活动|AIRDROP|70|空投
活动|MINE_REWARD|71|挖矿奖励

## 用户数据流接口

详细的用户信息流说明在另一个文档中。

### 开始用户信息流 (USER_STREAM)

```shell
POST /openapi/v1/userDataStream
```

开始一个新的用户信息流。如果keepalive指令没有发送，信息流将将会在60分钟后关闭。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### Keepalive用户信息流 (USER_STREAM)

```shell
PUT /openapi/v1/userDataStream
```

维持用户信息流来防止断开连接。用户信息流会在60分钟后自动中断，所以建议30分钟发送一次ping请求。

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```

### 关闭用户信息流 (USER_STREAM)

```shell
DELETE /openapi/v1/userDataStream
```

关闭用户信息流

**Weight:**
1

**Parameters:**

名称 | 类型 | 是否强制 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```
## 过滤层

过滤层（filter）定义某个broker的某个symbol的交易规则
过滤层（filter）有两个大类：`symbol filters` 和 `broker filters`

### Symbol过滤层

#### PRICE_FILTER

`PRICE_FILTER` 定义某个symbol的`price` 精度. 一共有3个部分：

* `minPrice` 定义最小允许的 `price`/`stopPrice`
* `maxPrice` 定义最大允许的 `price`/`stopPrice`.
* `tickSize` 定义`price`/`stopPrice` 可以增加和减少的间隔。

如果要通过`price filter`要求，`price`/`stopPrice`必须满足：

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0

**/brokerInfo格式:**

```javascript
  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
```

#### LOT_SIZE

`LOT_SIZE` 过滤层定义某个symbol `quantity`(在拍卖行里又称为"lots"）的精度。 一共有三个部分：

* `minQty` 定义最小允许的  `quantity`/`icebergQty`
* `maxQty` 定义最大允许的  `quantity`/`icebergQty`
* `stepSize`定义`quantity`/`icebergQty`可以增加和减少的间隔。

如果要通过`lot size`要求，`quantity`/`icebergQty`必须满足:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0

**/brokerInfo格式:**

```javascript
  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

#### MIN_NOTIONAL

`MIN_NOTIONAL` 过滤层定义某个symbol的名义金额精度。一个订单的名义金额为 `price` * `quantity`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000"
  }
```

#### MAX_NUM_ORDERS

`MAX_NUM_ORDERS` 过滤层定义账户在某个symbol上的最大挂单数。请注意，此过滤层同时计算“算法”订单和普通订单。

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ORDERS",
    "limit": 25
  }
```

#### MAX_NUM_ALGO_ORDERS

`MAX_ALGO_ORDERS` 过滤层定义账户在某个symbol上的最大“算法”挂单数。“算法”订单包括`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`等订单类型。

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
```

#### ICEBERG_PARTS

`ICEBERG_PARTS` 过滤层定义冰山订单部件的最大值。`ICEBERG_PARTS`的定义为`CEIL(qty / icebergQty)`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
```

### Broker Filters

#### BROKER_MAX_NUM_ORDERS

`BROKER_MAX_NUM_ORDERS` 过滤层定义账户在broker上的最大挂单数。请注意，此过滤层同时计算“算法”订单和普通订单。

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ORDERS",
    "limit": 1000
  }
```

#### BROKER_MAX_NUM_ALGO_ORDERS

`BROKER_MAX_NUM_ALGO_ORDERS` 过滤层定义账户在broker上的最大“算法”挂单数。“算法”订单包括`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`等订单类型。

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ALGO_ORDERS",
    "limit": 200
  }
```