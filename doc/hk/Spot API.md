# 幣幣交易

## 術語解釋

* `base asset` 指的是symbol的`quantity`（即數量）。

* `quote asset` 指的是symbol的`price`（即價格）。

## ENUM 定義

**Symbol 狀態:**

* TRADING - 交易中
* HALT - 終止
* BREAK - 斷開

**Symbol 類型:**

* SPOT - 現貨

**資產類型:**

* CASH - 現金
* MARGIN - 保證金

**訂單狀態:**

* NEW - 新訂單，暫無成交
* PARTIALLY_FILLED - 部分成交
* FILLED - 完全成交
* CANCELED - 已取消
* PENDING_CANCEL - 等待取消
* REJECTED - 被拒絕

**訂單類型:**

* LIMIT - 限價單
* MARKET - 市價單
* LIMIT_MAKER - maker限價單
* STOP_LOSS (unavailable now) - 暫無
* STOP_LOSS_LIMIT (unavailable now) - 暫無
* TAKE_PROFIT (unavailable now) - 暫無
* TAKE_PROFIT_LIMIT (unavailable now) - 暫無
* MARKET_OF_PAYOUT (unavailable now) - 暫無

**訂單方向:**

* BUY - 買單
* SELL - 賣單

**訂單時效類型:**

* GTC
* IOC
* FOK

**k線/燭線圖區間:**

m -> 分鐘; h -> 小時; d -> 天; w -> 周; M -> 月

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

**頻率限制類型 (rateLimitType)**

* REQUESTS_WEIGHT
* ORDERS

**頻率限制區間**

* SECOND
* MINUTE
* DAY

## 通用接口

### 測試連接

```shell
GET /openapi/v1/ping
```

測試REST API的連接。

**Weight:**
0

**Parameters:**
NONE

**Response:**

```javascript
{}
```

### 伺服器時間

```shell
GET /openapi/v1/time
```

測試連接並獲取當前伺服器的時間。

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

### Broker資訊

```shell
GET /openapi/v1/brokerInfo
```

當前broker交易規則和symbol資訊

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

### 訂單簿

```shell
GET /openapi/quote/v1/depth
```

**Weight:**

根據limit不同：

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 默認 100; 最大 100.

**注意:** 如果設置limit=0會返回很多數據。

**Response:**

[價格, 數量]

```javascript
{
"bids": [
[
"3.90000000", // 價格
"431.00000000" // 數量
],
[
"4.00000000",
"431.00000000"
]
],
"asks": [
[
"4.00000200", // 價格
"12.00000000" // 數量
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

獲取當前最新成交（最多500）

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
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

### k線/燭線圖數據

```shell
GET /openapi/quote/v1/klines
```

symbol的k線/燭線圖數據
K線會根據開盤時間而辨別。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | 默認500; 最大1000.

* 如果startTime和endTime沒有發送，只有最新的K線會被返回。

**Response:**

```javascript
[
[
1499040000000, // 開盤時間
"0.01634790", // 開盤價
"0.80000000", // 最高價
"0.01575800", // 最低價
"0.01577100", // 收盤價
"148976.11427815", // 交易量
1499644799999, // 收盤時間
"2434.19055334", // Quote asset數量
308, // 交易次數
"1756.87402397", // Taker buy base asset數量
"28.46694368" // Taker buy quote asset數量
]
]
```

### 24小時ticker價格變化數據

```shell
GET /openapi/quote/v1/ticker/24hr
```

24小時價格變化數據。**注意** 如果沒有發送symbol，會返回很多數據。

**Weight:**

如果只有一個symbol，1; 如果symbol沒有被發送，**40**。

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol沒有被發送，所有symbol的數據都會被返回。

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

### Symbol價格

```shell
GET /openapi/quote/v1/ticker/price
```

單個或多個symbol的最新價。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol沒有發送，所有symbol的最新價都會被返回。

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

### Symbol最佳訂單簿價格

```shell
GET /openapi/quote/v1/ticker/bookTicker
```

單個或者多個symbol的最佳買單賣單價格。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 如果symbol沒有被發送，所有symbol的最佳訂單簿價格都會被返回。

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

## 帳戶接口

### 創建新訂單 (TRADE)

```shell
POST /openapi/v1/order (HMAC SHA256)
```

發送一個新的訂單

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
assetType | STRING | NO |
side | ENUM | YES |
type | ENUM | YES |
timeInForce | ENUM | NO |
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 一個自己給訂單定義的ID，如果沒有發送會自動生成。
stopPrice | DECIMAL | NO | 與 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 和`TAKE_PROFIT_LIMIT` 訂單一起使用. **當前不可用**
icebergQty | DECIMAL | NO | 與 `LIMIT`, `STOP_LOSS_LIMIT`, 和 `TAKE_PROFIT_LIMIT` 來創建冰山訂單. **當前不可用**
recvWindow | LONG | NO |
timestamp | LONG | YES |

在`type`上的額外強制參數:

類型 | 額外強制參數
------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`
`MARKET` | `quantity`
`STOP_LOSS` | `quantity`, `stopPrice` **當前不可用**
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **當前不可用**
`TAKE_PROFIT` | `quantity`, `stopPrice` **當前不可用**
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **當前不可用**
`LIMIT_MAKER` | `quantity`, `price`

**Response:**

```javascript
{
"orderId": 28,
"clientOrderId": "6k9M212T12092"
}
```

### 測試新訂單 (TRADE)

```shell
POST /openapi/v1/order/test (HMAC SHA256)
```

用signature和recvWindow測試生成新訂單。
創建和驗證一個新訂單但是不送入撮合引擎。

**Weight:**
1

**Parameters:**

和 `POST /openapi/v1/order`一樣。

**Response:**

```javascript
{}
```

### 查詢訂單 (USER_DATA)

```shell
GET /openapi/v1/order (HMAC SHA256)
```

查詢訂單狀態。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

Notes:

* 單一 `orderId` 或者 `origClientOrderId` 必須被發送。
* 對於某些歷史數據 `cummulativeQuoteQty` 可能會 < 0, 這說明數據當前不可用。

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

### 取消訂單 (TRADE)

```shell
DELETE /openapi/v1/order (HMAC SHA256)
```

取消當前正在交易的訂單。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
orderId | LONG | NO |
clientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

單一 `orderId` 或者 `clientOrderId`必須被發送。

**Response:**

```javascript
{
"symbol": "LTCBTC",
"clientOrderId": "tU721112KM",
"orderId": 1,
"status": "CANCELED"
}
```

### 當前訂單(USER_DATA)

```shell
GET /openapi/v1/openOrders (HMAC SHA256)
```

獲取當前單個或者多個symbol的當前訂單。**注意** 如果沒有發送symbol，會返回很多數據。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
limit | INT | NO | 默認 500; 最多 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`設定好了，會篩選訂單小於`orderId`的。否則會返回最近的訂單資訊。

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

### 歷史訂單 (USER_DATA)

```shell
GET /openapi/v1/historyOrders (HMAC SHA256)
```
獲取當前帳戶的所有訂單。亦或是取消的，完全成交的，拒絕的。

**Weight:**
5

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
symbol | String | NO |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`設定好了，會篩選訂單小於`orderId`的。否則會返回最近的訂單資訊。

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

### 帳戶資訊 (USER_DATA)

```shell
GET /openapi/v1/account (HMAC SHA256)
```

獲取當前帳戶資訊

**Weight:**
5

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
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

### 帳戶交易記錄 (USER_DATA)

```shell
GET /openapi/v1/myTrades (HMAC SHA256)
```

獲取當前帳戶歷史成交記錄

**Weight:**
5

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from.
toId | LONG | NO | TradeId to fetch to.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果只有`fromId`，會返回訂單號小於`fromId`的，倒序排列。
* 如果只有`toId`，會返回訂單號小於`toId`的，昇冪排列。
* 如果同時有`fromId`和`toId`, 會返回訂單號在`fromId`和`toId`的，倒序排列。
* 如果`fromId`和`toId`都沒有，會返回最新的成交記錄，倒序排列。
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

### 帳戶存款記錄 (USER_DATA)

```shell
GET /openapi/v1/depositOrders (HMAC SHA256)
```

獲取當前帳戶的存款記錄

**Weight:**
5

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | 從哪個OrderId起開始抓取。默認抓取最新的存款記錄。
limit | INT | NO | 默認 500; 最大 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**

* 如果`orderId`設定好了，會篩選訂單小於`orderId`的。否則會返回最近的訂單資訊。

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

### 子帳戶列表(SUB_ACCOUNT_LIST)

```shell
POST /openapi/v1/subAccount/query
```

查詢子帳戶列表

**Parameters:**

無

**Weight:**
5

**Response:**

```javascript
[
{
"accountId": "122216245228131",
"accountName": "",
"accountType": 1,
"accountIndex": 0 // 帳戶index 0 默認帳戶 >0, 創建的子帳戶
},
{
"accountId": "482694560475091200",
"accountName": "createSubAccountByCurl", // 子帳戶名稱
"accountType": 1, // 子帳戶類型 1 幣幣帳戶 3 合約帳戶
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

### 帳戶內轉賬 (ACCOUNT_TRANSFER)

```shell
POST /openapi/v1/transfer
```

轉賬

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
fromAccountType | int | YES |源帳戶類型, 1 錢包(幣幣)帳戶 2 期權帳戶 3 合約帳戶
fromAccountIndex | int | YES |子帳戶index, 主帳戶Api調用時候有用，從子帳戶列表介面獲取
toAccountType | int | YES | 目標帳戶類型, 1 錢包(幣幣)帳戶 2 期權帳戶 3 合約帳戶
toAccountIndex | int | YES | 子帳戶index, 主帳戶Api調用時候有用，從子帳戶列表介面獲取
tokenId | STRING | YES | tokenID
amount | STRING | YES | 轉賬數量

**Response:**

```javascript
{
"success":"true" // 0成功
}
```

**說明**

1、轉賬帳戶和收款帳戶的其中一方，必須是主帳戶(錢包帳戶)

2、主帳戶Api可以從錢包帳戶向其他帳戶(包括子帳戶)轉賬，也可以從其他帳戶向錢包帳戶轉賬

3、**子帳戶Api調用的時候只能從當前子帳戶向主帳戶(錢包帳戶)轉賬，所以fromAccountType\fromAccountIndex\toAccountType\toAccountIndex不用填**

### 查詢流水 (BALANCE_FLOW)

```shell
POST /openapi/v1/balance_flow
```

查詢帳戶流水

**Weight:**
5

**Parameters:**

|名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
| accountType | int | 否 | 帳戶對應的account_type | 默認1 |
| accountIndex | int | 否 | 帳戶對應的account_index | 默認0 |
| tokenId | string | 否 | token_id | eg: BTC |
| fromFlowId | long | 否 | 順向查詢數據 | 指定查詢 id < fromFlowId的數據 |
| endFlowId | long | 否 | 反向查詢數據 | 指定查詢 id > |endFlowId的數據 |
| startTime | long | 否 | 開始時間 | 毫秒時間戳 |
| endTime | long | 否 | 結束時間 | 毫秒時間戳 |
| limit | integer | 否 | 每頁記錄數 | 默認50，最大100 |

**Response:**

```javascript
[
{
"id": "539870570957903104",
"accountId": "122216245228131",
"tokenId": "BTC",
"tokenName": "BTC",
"flowTypeValue": 51, // 流水類型
"flowType": "USER_ACCOUNT_TRANSFER", // 流水類型名稱
"flowName": "Transfer", // 流水類型說明
"change": "-12.5", // 變動值
"total": "379.624059937852365", // 變動後當前tokenId總資產
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

**說明**

1、主帳戶Api可以查詢錢包帳戶或者其他帳戶(包括子帳戶，指定accountType和accountIndex)的流水’

2、子帳戶Api只能查詢當前子帳戶的流水，所以不用指定accountType和accountIndex

**流水類型說明請見如下**

歸類|類型參數名|類型參數代號|解釋說明|
------|------|------|------|
通用流水類|TRADE|1|交易|
通用流水類|FEE|2|交易手續費|
通用流水類|TRANSFER|3|轉賬|
通用流水類|DEPOSIT|4|充值|
衍生品業務|MAKER_REWARD|27|maker獎勵
衍生品業務|PNL|28|期貨等的盈虧
衍生品業務|SETTLEMENT|30|交割
衍生品業務|LIQUIDATION|31|強平
衍生品業務|FUNDING_SETTLEMENT|32|期貨等的資金費率結算
用戶子帳戶之間內部轉賬|USER_ACCOUNT_TRANSFER|51|userAccountTransfer 專用，流水沒有subjectExtId
OTC|OTC_BUY_COIN|65|OTC 買入coin
OTC|OTC_SELL_COIN|66|OTC 賣出coin
OTC|OTC_FEE|73|OTC 手續費
OTC|OTC_TRADE|200|舊版 OTC 流水
活動|ACTIVITY_AWARD|67|活動獎勵
活動|INVITATION_REFERRAL_BONUS|68|邀請返傭
活動|REGISTER_BONUS|69|註冊送禮
活動|AIRDROP|70|空投
活動|MINE_REWARD|71|挖礦獎勵

## 用戶數據流接口

詳細的用戶資訊流說明在另一個文檔中。

### 開始用戶資訊流 (USER_STREAM)

```shell
POST /openapi/v1/userDataStream
```

開始一個新的用戶資訊流。如果keepalive指令沒有發送，資訊流將將會在60分鐘後關閉。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{
"listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### Keepalive用戶資訊流 (USER_STREAM)

```shell
PUT /openapi/v1/userDataStream
```

維持用戶資訊流來防止斷開連接。用戶資訊流會在60分鐘後自動中斷，所以建議30分鐘發送一次ping請求。

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```

### 關閉用戶資訊流 (USER_STREAM)

```shell
DELETE /openapi/v1/userDataStream
```

關閉用戶資訊流

**Weight:**
1

**Parameters:**

名稱 | 類型 | 是否強制 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**

```javascript
{}
```

## 過濾層

過濾層（filter）定義某個broker的某個symbol的交易規則
過濾層（filter）有兩個大類：`symbol filters` 和 `broker filters`

### Symbol過濾層

#### PRICE_FILTER

`PRICE_FILTER` 定義某個symbol的`price` 精度. 一共有3個部分：

* `minPrice` 定義最小允許的 `price`/`stopPrice`
* `maxPrice` 定義最大允許的 `price`/`stopPrice`.
* `tickSize` 定義`price`/`stopPrice` 可以增加和減少的間隔。

如果要通過`price filter`要求，`price`/`stopPrice`必須滿足：

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

`LOT_SIZE` 過濾層定義某個symbol `quantity`(在拍賣行裏又稱為"lots"）的精度。 一共有三個部分：

* `minQty` 定義最小允許的 `quantity`/`icebergQty`
* `maxQty` 定義最大允許的 `quantity`/`icebergQty`
* `stepSize`定義`quantity`/`icebergQty`可以增加和減少的間隔。

如果要通過`lot size`要求，`quantity`/`icebergQty`必須滿足:

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

`MIN_NOTIONAL` 過濾層定義某個symbol的名義金額精度。一個訂單的名義金額為 `price` * `quantity`.

**/brokerInfo format:**

```javascript
{
"filterType": "MIN_NOTIONAL",
"minNotional": "0.00100000"
}
```

#### MAX_NUM_ALGO_ORDERS

`MAX_ALGO_ORDERS` 過濾層定義賬戶在某個symbol上的最大“算法”掛單數。“算法”訂單包括`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`等訂單類型。

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
```

#### ICEBERG_PARTS

`ICEBERG_PARTS` 過濾層定義冰山訂單部件的最大值。`ICEBERG_PARTS`的定義為`CEIL(qty / icebergQty)`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
```

### Broker Filters

#### BROKER_MAX_NUM_ORDERS

`BROKER_MAX_NUM_ORDERS` 過濾層定義賬戶在broker上的最大掛單數。請註意，此過濾層同時計算“算法”訂單和普通訂單。

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ORDERS",
    "limit": 1000
  }
```

#### BROKER_MAX_NUM_ALGO_ORDERS

`BROKER_MAX_NUM_ALGO_ORDERS` 過濾層定義賬戶在broker上的最大“算法”掛單數。“算法”訂單包括`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`等訂單類型。

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ALGO_ORDERS",
    "limit": 200
  }
```