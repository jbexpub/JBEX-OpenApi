# 公有Broker Rest API (2020-01-25)

## 通用API資訊

* 所有的端點都會返回一個JSON object或者array.
* 數據返回的是一個 **昇冪**。更早的在前，更新的在後。
* 所有的時間/時間戳有關的變數都是milliseconds（毫秒級）。
* HTTP `4XX` 返回錯誤碼是指請求內容有誤，這個問題是在請求發起者這邊。
* HTTP `429` 返回錯誤碼是指請求次數上限被打破。
* HTTP `418` 返回錯誤碼是指IP在收到`429`錯誤碼後還繼續發送請求被自動封禁。
* HTTP `5XX` 返回錯誤碼是內部系統錯誤；這說明這個問題是在券商這邊。在對待這個錯誤時，**千萬** 不要把它當成一個失敗的任務，因為執行狀態 **未知**，有可能是成功也有可能是失敗。
* 任何端點都可能返回ERROR（錯誤）； 錯誤的返回payload如下

```javascript
{
"code": -1121,
"msg": "Invalid symbol."
}
```

* 詳細的錯誤碼和錯誤資訊在請見錯誤碼檔。
* 對於`GET`端點，必須發送參數為`query string`（查詢字串）。
* 對於`POST`, `PUT`, 和 `DELETE` 端點，必需要發送參數為`query string`（查詢字串）或者發送參數在`request body`（請求主體）並設置content type（內容類型）為`application/x-www-form-urlencoded`。可以同時在`query string`或者`request body`裏混合發送參數如果有需要的話。
* 參數可以以任意順序發送。
* 如果有參數同時在`query string` 和 `request body`裏存在，只有`query string`的參數會被使用。

### 限制

* 在 `/openapi/v1/brokerInfo`的`rateLimits` array裏存在當前broker的`REQUEST_WEIGHT`和`ORDER`頻率限制。
* 如果任一頻率限額被超過，`429` 會被返回。
* 每條線路有一個`weight`特性，這個決定了這個請求佔用多少容量（比如`weight`=2說明這個請求佔用兩個請求的量）。返回數據多的端點或者在多個symbol執行任務的端點可能有更高的`weight`。
* 當`429`被返回後，你有義務停止發送請求。
* **多次反復違反頻率限制和/或者沒有在收到429後停止發送請求的用戶將會被收到封禁IP（錯誤碼418）**
* IP封禁會被跟蹤和 **調整封禁時長**（對於反復違反規定的用戶，時間從 **2分鐘到3天不等**）

### 端點安全類型

* 每個端點有一個安全類型，這決定了你會怎麼跟其交互。
* API-key要以`X-BH-APIKEY`的名字傳到REST API header裏面。
* API-keys和secret-keys **要區分大小寫**。
* 默認情況下，API-keys可以訪問所有的安全節點。

安全類型 | 描述
------------ | ------------
NONE | 端點可以自由訪問。
TRADE | 端點需要發送有效的API-Key和簽名。
USER_DATA | 端點需要發送有效的API-Key和簽名。
USER_STREAM | 端點需要發送有效的API-Key。
MARKET_DATA | 端點需要發送有效的API-Key。

* `TRADE` 和 `USER_DATA` 端點是 `SIGNED`（需要簽名）的端點。

### SIGNED（有簽名的）(TRADE和USER_DATA) 端點安全

* `SIGNED`（需要簽名）的端點需要發送一個參數，`signature`，在`query string` 或者 `request body`裏。
* 端點用`HMAC SHA256`簽名。`HMAC SHA256 signature`是一個對key進行`HMAC SHA256`加密的結果。用你的`secretKey`作為key和`totalParams`作為value來完成這一加密過程。
* `signature` **不區分大小寫**。
* `totalParams` 是指 `query string`串聯`request body`。

### 時效安全

* 一個`SIGNED`(有簽名)的端點還需要發送一個參數，`timestamp`，這是當請求發起時的毫秒級時間戳。
* 一個額外的參數（非強制性）, `recvWindow`, 可以說明這個請求在多少毫秒內是有效的。如果`recvWindow`沒有被發送，**默認值是5000**。
* 在當前，只有創建訂單的時候才會用到`recvWindow`。
* 該參數的邏輯如下：

```javascript
if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
// process request
} else {
// reject request
}
```

**嚴謹的交易和時效緊緊相關** 網路有時會不穩定或者不可靠，這會導致請求發送伺服器的時間不一致。
有了`recvWindow`，你可以說明在多少毫秒內請求是有效的，否則就會被服務器拒絕。

**建議使用一個相對小的recvWindow（5000或以下）！**

### SIGNED（簽名） 的例子（對於POST /openapi/v1/order）

這裏有一個詳細的用Linux`echo`, `openssl`, 和 `curl`舉例來展示如何發送一個有效的簽名payload。

Key | 值
------------ | ------------
apiKey | tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW
secretKey | lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76

參數名 | 參數值
------------ | ------------
symbol | ETHBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1538323200000

#### 例子 1: 在`queryString`裏

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/openapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### 例子 2: 在`request body`裏

* **requestBody:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/openapi/v1/order' -d 'symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=5f2750ad7589d1d40757a55342e621a44037dad23b5128cc70e18ec1d1c3f4c6'
```

#### 例子 3: `queryString`和`request body`混合在一起

* **queryString:** symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000
* **HMAC SHA256 signature:**

```shell
[linux]$ echo -n "symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000" | openssl dgst -sha256 -hmac "lH3ELTNiFxCQTmi9pPcWWikhsjO04Yoqw3euoHUuOLC3GYBW64ZqzQsiOEHXQS76"
(stdin)= 885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa
```

* **curl command:**

```shell
(HMAC SHA256)
[linux]$ curl -H "X-BH-APIKEY: tAQfOrPIZAhym0qHISRt8EFvxPemdBm5j5WMlkm3Ke9aFp0EGWC2CGM8GHV4kCYW" -X POST 'https://$HOST/openapi/v1/order?symbol=ETHBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1538323200000&signature=885c9e3dd89ccd13408b25e6d54c2330703759d7494bea6dd5a3d1fd16ba3afa'
```

***注意在例子3裏有一點不一樣，"GTC"和"quantity=1"之間沒有&。***

## 公共 API 端點

### 術語解釋

* `base asset` 指的是symbol的`quantity`（即數量）。

* `quote asset` 指的是symbol的`price`（即價格）。

### ENUM 定義

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

### 通用端點

#### 測試連接

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

#### 伺服器時間

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

#### Broker資訊

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

### 市場數據端點

#### 訂單簿

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

#### 最近成交

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

#### k線/燭線圖數據

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

#### 24小時ticker價格變化數據

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

#### Symbol價格

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

#### Symbol最佳訂單簿價格

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

### 帳戶端點

#### 創建新訂單 (TRADE)

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

#### 測試新訂單 (TRADE)

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

#### 查詢訂單 (USER_DATA)

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

#### 取消訂單 (TRADE)

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

#### 當前訂單(USER_DATA)

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

#### 歷史訂單 (USER_DATA)

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

#### 帳戶資訊 (USER_DATA)

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

#### 帳戶交易記錄 (USER_DATA)

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

#### 帳戶存款記錄 (USER_DATA)

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

### 用戶數據流端點

詳細的用戶資訊流說明在另一個文檔中。

#### 開始用戶資訊流 (USER_STREAM)

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

#### Keepalive用戶資訊流 (USER_STREAM)

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

#### 關閉用戶資訊流 (USER_STREAM)

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

#### 子帳戶列表(SUB_ACCOUNT_LIST)

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

#### 帳戶內轉賬 (ACCOUNT_TRANSFER)

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

#### 查詢流水 (BALANCE_FLOW)

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

### 過濾層

過濾層（filter）定義某個broker的某個symbol的交易規則
過濾層（filter）有兩個大類：`symbol filters` 和 `broker filters`

#### Symbol過濾層

##### PRICE_FILTER

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

##### LOT_SIZE

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

##### MIN_NOTIONAL

`MIN_NOTIONAL` 過濾層定義某個symbol的名義金額精度。一個訂單的名義金額為 `price` * `quantity`.

**/brokerInfo format:**

```javascript
{
"filterType": "MIN_NOTIONAL",
"minNotional": "0.00100000"
}
```

