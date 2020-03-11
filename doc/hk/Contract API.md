# 合約交易

Broker Open API的地址請見[這裏](endpoint.md)

## 公共接口

### `brokerInfo`

獲取當前broker的交易規則和合約symbol的資訊（精度單位等資訊），包括合約的風險限額和乘數等資訊。

#### **Request Weight:**

0

#### **Request Url:**
```bash
GET /openapi/v1/brokerInfo
```

#### **Parameters：**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ----
`type`|string|`NO`|| 交易類型，支持的類型現為`token`（幣幣）、`options`（期權）、`contracts`（合約）。如果沒有發送此參數，所有交易類型的symbol資訊都會被返回。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`timezone`|string|`UTC`|時間戳的時區。
`serverTime`|long|`1554887652929`|返回現在的伺服器時間戳（毫秒級）

在`rateLimits`資訊組裏:
下單api的請求限制將會被展示。

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`rateLimitType`|string|`ORDERS`|速度限制類型
`interval`|string|`SECOND`|速度限制區間
`limit`|string|`1500`|速度限制區間價值

在`contracts`資訊組裏:
所有當前券商正在交易的合約的資訊將會被返回：

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-PERP-REV`|合約名稱
`status`|string|`TRADING`|合約狀態
`baseAsset`|string|`BTC-PERP-REV`|基礎資產。對於合約來說，合約本身就是基礎資產。
`baseAssetPrecision`|float|`0.001`|基礎資產（合約數量）的精度
`quoteAsset`|string|`USDT`|定價資產。對於合約來說，這個是合約是以什麼來結算的。
`quoteAssetPrecision`|float|`0.001`|定價資產（合約價格）的精度。
`inverse`|bool|`true`|合約是否為反向合約（true=是反向合約，false=是正向合約）。
`index`|string|`BTCUSDT`|標的指數的名稱。標的指數即時價格可在`index`端點訪問得到。比如`BTC-PERP-REV`使用`BTCUSDT`為標的指數，那麼可以在`index`端點尋找`BTCUSDT`的即時價格。
`contractMultiplier`|string|`true`|合約的乘數。
`icebergAllowed`|string|`false`|是否支持冰山訂單。

在 `contracts`的`filters`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`filterType`|string|`PRICE_FILTER`|篩檢程式類型。
`minPrice`|float|`0.001`|允許的最小價格。
`maxPrice`|float|`100000.00000000`|允許的最大價格。
`tickSize`|float|`0.001`|合約價格的精度。
`minQty`|float|`0.01`|允許的最小數量。
`maxQty`|float|`100000.00000000`|允許的最大數量。
`stepSize`|float|`0.001`|合約數量的精度。
`minNotional`|float|`1`|最小交易額限制(數量 * 價格)

在`riskLimits`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`quantity`|float|`100`|倉位小於此數且大於前一個檔位的`quantity`的需要參照以下要求。
`initialMargin`|float|`0.1`|初始保證金率。
`maintMargin`|float|`0.03`|最小維持保證金率。

#### **Example:**
```js
{
"timezone":"UTC",
"serverTime":"1570701444309",
"brokerFilters":[],
"rateLimits":[
{
"rateLimitType":"ORDERS",
"interval":"SECOND",
"limit":20
},
{"rateLimitType":"ORDERS",
"interval":"DAY",
"limit":350000
},{
"rateLimitType":"REQUEST_WEIGHT",
"interval":"MINUTE",
"limit":1500
}],
"contracts":[
{
"filters":[
{"minPrice":"0.01",
"maxPrice":"100000.00000000",
"tickSize":"0.01",
"filterType":"PRICE_FILTER"
},
{
"minQty":"1",
"maxQty":"100000.00000000",
"stepSize":"1",
"filterType":"LOT_SIZE"
},{
"minNotional":"0.000001",
"filterType":"MIN_NOTIONAL"
}],
"exchangeId":"301",
"symbol":"BTC-PERP-REV",
"symbolName":"BTC-PERP-REV",
"status":"TRADING",
"baseAsset":"BTC-PERP-REV",
"baseAssetPrecision":"1",
"quoteAsset":"USDT",
"quoteAssetPrecision":"0.01",
"icebergAllowed":false,
"inverse":true,
"index":"BTCUSDT",
"marginToken":"TBTC",
"marginPrecision":"0.00000001",
"contractMultiplier":"1.0",
"riskLimits":[
{
"riskLimitId":"200000001",
"quantity":"1000000.0",
"initialMargin":"0.01",
"maintMargin":"0.005"
},
{
"riskLimitId":"200000002",
"quantity":"2000000.0",
"initialMargin":"0.02",
"maintMargin":"0.01"
},
{
"riskLimitId":"200000003",
"quantity":"3000000.0",
"initialMargin":"0.03",
"maintMargin":"0.015"
},
{
"riskLimitId":"200000004",
"quantity":"4000000.0",
"initialMargin":"0.04",
"maintMargin":"0.02"
}
]
}
]
}
```

<!-- ### `insurance` **(PENDING)**
獲取當前保險基金資訊。

#### **Request Weight:**
0

#### **Request Url:**
```bash
GET /openapi/contract/v1/insurance
```

#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ------------
`symbol`|string|`NO`||Input specific symbol to return the corresponding records. If not entered, records for all symbols will be returned.
`fromId`|long|`NO`||pagination, return record which id < fromId
`toId`|long|`NO`||pagination, return record which id > toId. If toId is given, toId cannot be 0.
`limit`|integer|`NO`|20|Number of entries returned.

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`id`|long|`1552272184833`|The ID of the record.
`timestamp`|long|`155227218483`|current server timestamp.
`value`|float|`23.23`|Balance of the insurance fund.
`unit`|string|`BTC`|Unit of the balance..

```js
{
"BTC":[
{
"id": 1552272184833,
"timestamp": 155227218483,
"value": 23.23,
"unit": "BTC"
},...
],...
}
``` -->

### `index`

標的指數價格

#### **Request Weight:**
0

#### **Request Url:**
```bash
GET /openapi/quote/v1/contract/index
```
#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ----
symbol|string|`NO`||標的指數名稱。如果這個沒有發送，所有標的指數的價格都會被返回。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`index`|float|`8342.73`|標的指數的價格。
`EDP`|float|`8432.32`|標的指數的EDP(預估交割價，過去10分鐘指數價格的平均值）。

#### **Example:**
```js
{
"index":{
"BTCUSDT":"8243.21666667",
"OKBUSDT":"1.482",
"BNBUSDT":"31.2658",
"HTUSDT":"3.1209",...
},
"edp":{
"BTCUSDT":"8258.98505556",
"OKBUSDT":"1.48578333",
"BNBUSDT":"31.48741917",
"HTUSDT":"3.14308",...
}
}
```

### `fundingRate`

獲取當前資金費率 (*歷史資金費率正在建設*)

#### **Request Weight:**
0

#### **Request Url:**
```bash
GET /openapi/contract/v1/fundingRate
```

#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ----
`symbol`|string|`NO`||合約名稱。如果沒有發送該參數，所有合約的資金費率都會被返回。
`state`|string|`NO`|`current`|獲取`current`（當前）或者`past`（歷史）的資金費率。
`from`|long|`NO`||開始時間戳。
`to`|long|`NO`||結束時間戳。
`limit`|integer|`NO`|`20`| 返回條數。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-PERP-REV`|合約名稱。
`intervalStart`|long|`1554710400000`|本次結算開始時間。
`intervalEnd`|long|`1554710400000`|本次結算結束時間。
`rate`|float|`0.00321`|該次結算資金費率。

#### **Example:**

```js
[
{
'symbol': 'BTC-PERP-REV',
'intervalStart': '1570708800000',
'intervalEnd': '1570737600000',
},...
]
```

### `depth`

獲取當前訂單簿的數據。

#### **Request Weight:**

根據數量會不一樣，請求數量越多，重量越大:

數量|請求重量
------------ | ------------
5, 10, 20, 50, 100|1
500|5
1000|10

#### **Request Url:**
```
GET /openapi/quote/v1/contract/depth
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string|`YES`||用來獲取訂單簿的合約名稱。
`limit`|integer|`NO`|`100` (max = 100)|返回`bids`和`asks`的數量

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1550829103981`|當前時間（Unix Timestamp，毫秒ms）
`bids`|list|(如下)|所有bid的價格和數量資訊，最優bid價格由上到下排列。
`asks`|list|(如下)|所有ask的價格和數量資訊，最優ask價格由上到下排列。

`bids`和`asks`所對應的資訊組代表了訂單簿的所有價格以及價格對應數量的資訊，由最優價格從上到下排列。

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`''`|float|`123.10`|價格
`''`|float|`300`|當前價格對應的數量

#### **Example:**

```js
{
"time": 1555049455783,
"bids": [
["78.82", "0.526"],//[價格, 數量]
["77.24", "1.22"],
["76.65", "1.043"],
["76.58", "1.34"],
["75.67", "1.52"],
["75.12", "0.635"],
["75.02", "0.72"],
["75.01", "0.672"],
["73.73", "1.282"],
["73.58", "1.116"],
["73.45", "0.471"],
["73.44", "0.483"],
["72.32", "0.383"],
["72.26", "1.283"],
["72.11", "0.703"],
["70.61", "0.454"]],
"asks": [
["122.96", "0.381"],//[價格, 數量]
["144.46", "1"],
["155.55", "0.065"],
["160.16", "0.052"],
["200", "0.775"],
["249", "0.17"],
["250", "1"],
["300", "1"],
["400", "1"],
["499", "1"]]
}

```

### `trades`

獲取某個合約最近成交訂單的資訊。

#### **Request Weight:**

1

#### **Request URL:**
```
GET /openapi/quote/v1/contract/trades
```
#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ------
`symbol`|string|`YES`||合約名稱
`limit`|integer|`NO` (最大值為1000)|`100`|返回成交訂單的數量

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`price`|float|`0.055`|交易價格
`time`|long|`1537797044116`|當前Unix時間戳，毫秒(ms)
`qty`|float|`5`|數量（張數）
`isBuyerMaker`|string|`true`|賣方還是買方。`true`=賣方，`false`=買方

#### **Example:**
```js
[
{
"price": "1.21",
"time": 1555034474064,
"qty": "0.725",
"isBuyerMaker": False
},...
]
```

### `klines`

獲取某個合約的K線資訊（高，低，開，收，交易量...)

#### **Request Weight:**

1

#### **Request URL:**
```
GET /openapi/quote/v1/contract/klines
```

#### **Parameters：**
| 名稱|類型|是否強制|默認|描述 |
| ------------ | ------------ | ------------ | ------------ | ---- |
`symbol`|string|`YES`||合約名稱
`interval`|string|`YES`||K線圖區間。可識別發送的值為： `1m`,`5m`,`15m`,`30m`,`1h`,`1d`,`1w`,`1M`（`m`=分鐘，`h`=小時,`d`=天，`w`=星期，`M`=月）
`limit`|integer|`NO`|`1000`|返回值的數量，最大值為1000
`to`|integer|`NO`||最後一個數據點的時間戳

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`''`|long|`1538728740000`|開始時間戳，毫秒（ms）
`''`|float|`36.00000'`|開盤價
`''`|float|`36.00000`|最高價
`''`|float|`36.00000`|最低價
`''`|float|`36.00000`|收盤價
`''`|float|`148976.11427815`|合約交易金額
`''`|long|`1538728740000`|停止時間戳，毫秒（ms）
`''`|float|`2434.19055334`|交易數量（張數）
`''`|integer|`308`|已成交數量（張數）
`''`|float|`1756.87402397`|買方購買金額
`''`|float|`28.46694368`|買方購買數量（張數）

#### **Example:**
```js
[
[
1538728740000, //'開盤時間'
'36.000000000000000000', //'開盤價'
'36.000000000000000000', //'最高價'
'36.000000000000000000', //'最低價':
'36.000000000000000000', //'收盤價'
'148976.11427815', // 合約交易金額
1499644799999, // 收盤時間
'2434.19055334', // 交易數量（張數）
308, // 已成交數量（張數）
'1756.87402397', // 買方購買金額
'28.46694368' // 買方購買數量（張數）
],...
]
```
`base asset` 代表買方收到了該數量的合約

`quote asset` 代表該數量的token用來獲得合約

## 私有接口

### `order`

合約下單，這個端點需要簽名訪問。

#### **Request Weight:**

1

#### **Request URL:**
```bash
POST /openapi/contract/v1/order
```

#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ------------
`symbol`|string|`YES`||合約名稱。
`side`|string|`YES`||下單方向，方向類型為`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`。
`orderType`|string|`YES`||訂單類型，支持訂單類型為 `LIMIT`和`STOP`。
`quantity`|float|`YES`||訂單的合約數量。
`leverage`|float|`YES`.（\*\_CLOSE平倉單**不強制**）||訂單的杠杆。
`price`|float|`NO`. (`LIMIT`&`INPUT`)訂單 **強制需要** ||訂單價格。
`priceType`|string|`NO`|`INPUT`|價格類型，支持的價格類型為`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`。
`triggerPrice`|float|`NO`. `STOP` 訂單 **強制需要** ||計畫委托的觸發價格。
`timeInForce`|string|`NO`|`GTC`|`LIMIT`訂單的時間指令（Time in Force），目前支持的類型為`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`。
`clientOrderId`|string/long|`YES`||訂單的ID，用戶自己定義。

**注意：** 對於 **市價訂單**, 你需要設置`orderType`為 **`LIMIT`** **並且設置** `priceType` 為 **`MARKET`**.

你可以在`brokerInfo`端點獲取合約價格，數量的精度配置資訊。

注意：如果你的餘額沒有達到需要保證金的要求（初始保證金+開倉手續費+平倉手續費），將會有"*insufficient balance*"（餘額不足）的錯誤返回。

對於 *價格類型* 和 *訂單類型* 的詳細解釋，請參見文末。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|訂單生成時的時間戳
`updateTime`|long|`1551062936784`|訂單上次更新的時間戳
`orderId`|integer|`469961015902208000`|訂單ID
`clientOrderId`|string|`213443`|用戶定義的訂單ID
`symbol`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`8200`|訂單價格
`leverage`|float|`4`|訂單杠杆
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|訂單已執行數量
`avgPrice`|float|`4754.24`|平均交易價格
`marginLocked`|float|`200`|該訂單鎖定的保證金。這包括實際需要的保證金外加開倉和平倉所需的費用。
`orderType`|string|`YES`|訂單類型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|價格類型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|訂單狀態（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`timeInForce`|string|`GTC`|時效單（Time in Force)類型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||訂單的手續費

#### **Example:**
```js
{
'time': '1570759718825',
'updateTime': '0',
'orderId': '469961015902208000',
'clientOrderId': '6423344174',
'symbol': 'BTC-PERP-REV',
'price': '8200',
'leverage': '12.08',
'origQty': '5',
'executedQty': '0',
'avgPrice': '0',
'marginLocked': '0.00005047',
'orderType': 'LIMIT',
'side': 'BUY_OPEN',
'fees': [],
'timeInForce': 'GTC',
'status': 'NEW',
'priceType': 'INPUT'
}
```

### `cancel`

取消某個訂單，需要發送`orderId`或者`clientOrderId`其中之一。這個API端點需要你的請求簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
DELETE /openapi/contract/v1/order/cancel
```

#### **Parameter:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -----
`orderId`|integer|`NO`||訂單的ID
`clientOrderId`|string|`NO`||用戶定義的訂單ID
`orderType`|string|`YES`||訂單類型（`LIMIT`和`STOP`）

**注意：**` orderId` 或者 `clientOrderId` **必須發送其中之一**

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|訂單生成時的時間戳
`updateTime`|long|`1551062936784`|訂單上次更新的時間戳
`orderId`|integer|`469961015902208000`|訂單ID
`clientOrderId`|string|`213443`|用戶定義的訂單ID
`symbol`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`8200`|訂單價格
`leverage`|float|`4`|訂單杠杆
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|訂單已執行數量
`avgPrice`|float|`4754.24`|平均交易價格
`marginLocked`|float|`200`|該訂單鎖定的保證金。這包括實際需要的保證金外加開倉和平倉所需的費用。
`orderType`|string|`YES`|訂單類型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|價格類型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|訂單狀態（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）。該端點返回的訂單狀態都是`CANCELED`
`timeInForce`|string|`GTC`|時效單（Time in Force)類型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||訂單的手續費

在`fees`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價類型
`fee`|float|`0`|實際手續費

#### **Example:**

```js
{
'time': '1570759718825',
'updateTime': '0',
'orderId': '469961015902208000',
'clientOrderId': '6423344174',
'symbol': 'BTC-PERP-REV',
'price': '8200',
'leverage': '12.08',
'origQty': '5',
'executedQty': '0',
'avgPrice': '0',
'marginLocked': '0',
'orderType': 'LIMIT',
'side': 'BUY_OPEN',
'fees': [],
'timeInForce': 'GTC',
'status': 'CANCELED',
'priceType': 'INPUT' //status will always be `CANCELED` for cancel request
}
```

#### `batchCancel`

批量撤銷訂單(**正在建設中: 計畫委托的批量撤單**)

#### **Request Weight:**

1

#### **Request Url:**
```bash
DELETE /openapi/contract/v1/order/batchCancel
```
#### **Parameter:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string/list|`NO`||合約名稱（或者用`,`分割的合約名稱的list）

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`message`|string|`success`|撤單請求的返回消息
`timestamp`|long|`1541161088303`|返回時的時間戳

#### **Example:**

```js
{
'message':'success',
'timestamp':1541161088303
}
```

#### `openOrders`

未完成委託，這個API端點需要請求簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/contract/v1/openOrders
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||合約名稱。如果沒有在請求裏發送，所有合約的未完成委託都會被返回。
`orderId`|integer|`NO`||訂單ID
`side`|string|`NO`||訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`orderType`|string|`YES`||訂單類型（`LIMIT`、`STOP`）
`limit`|integer|`NO`|`20`|返回值的長度。

如果發送了`orderId`，會返回所有< `orderId`的訂單。如果沒有則會返回最新的未完成訂單。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|訂單生成時的時間戳
`updateTime`|long|`1551062936784`|訂單上次更新的時間戳
`orderId`|integer|`469961015902208000`|訂單ID
`clientOrderId`|string|`213443`|用戶定義的訂單ID
`symbol`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`8200`|訂單價格
`leverage`|float|`4`|訂單杠杆
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|訂單已執行數量
`avgPrice`|float|`4754.24`|平均交易價格
`marginLocked`|float|`200`|該訂單鎖定的保證金。這包括實際需要的保證金外加開倉和平倉所需的費用。
`orderType`|string|`YES`|訂單類型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|價格類型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|訂單狀態（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`timeInForce`|string|`GTC`|時效單（Time in Force)類型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||訂單的手續費

在`fees`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價類型
`fee`|float|`0`|實際手續費

#### **Example:**

```js
[
{
'time': '1570760254539',
'updateTime': '0',
'orderId': '469965509788581888',
'clientOrderId': '1570760253946',
'symbol': 'BTC-PERP-REV',
'price': '8502.34',
'leverage': '20',
'origQty': '222',
'executedQty': '0',
'avgPrice': '0',
'marginLocked': '0.00130552',
'orderType': 'LIMIT',
'side': 'BUY_OPEN',
'fees': [],
'timeInForce': 'GTC',
'status': 'NEW',
'priceType': 'INPUT'
},...
]
```

### `historyOrders`

Retrieves history of orders that have been partially or fully filled or canceled. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/contract/v1/historyOrders
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||Symbol to return open orders for. If not sent, orders of all contracts will be returned.
`orderId`|integer|`NO`|| Order ID
`side`|string|`NO`||Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`.
`orderType`|string|`YES`||The order type, possible types: `LIMIT`, `STOP`
`limit`|integer|`NO`|`20`|Number of entries to return.

If `orderId` is set, it will get orders < that `orderId`. Otherwise most recent orders are returned.

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|訂單生成時的時間戳
`updateTime`|long|`1551062936784`|訂單上次更新的時間戳
`orderId`|integer|`469961015902208000`|訂單ID
`clientOrderId`|string|`213443`|用戶定義的訂單ID
`symbol`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`8200`|訂單價格
`leverage`|float|`4`|訂單杠杆
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|訂單已執行數量
`avgPrice`|float|`4754.24`|平均交易價格
`marginLocked`|float|`200`|該訂單鎖定的保證金。這包括實際需要的保證金外加開倉和平倉所需的費用。
`orderType`|string|`YES`|訂單類型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|價格類型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|訂單狀態（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`timeInForce`|string|`GTC`|時效單（Time in Force)類型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||訂單的手續費

在`fees`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價類型
`fee`|float|`0`|實際手續費

#### **Example:**

```js
[
{
'time': '1570759718825',
'updateTime': '0',
'orderId': '469961015902208000',
'clientOrderId': '6423344174',
'symbol': 'BTC-PERP-REV',
'price': '8200',
'leverage': '12.08',
'origQty': '5',
'executedQty': '0',
'avgPrice': '0',
'marginLocked': '0',
'orderType': 'LIMIT',
'side': 'BUY_OPEN',
'fees': [],
'timeInForce': 'GTC',
'status': 'CANCELED',
'priceType': 'INPUT'
},...
]
```

### `getOrder`

獲取某個訂單的詳細資訊

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/contract/v1/getOrder
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`orderId`|integer|`NO`||訂單ID
`clientOrderId`|string|`NO`||用戶定義的訂單ID
`orderType`|string|`YES`||訂單類型（`LIMIT`和`STOP`）

**注意：**` orderId` 或者 `clientOrderId` **必須發送其中之一**

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1570759718825`|訂單生成時的時間戳
`updateTime`|long|`1551062936784`|訂單上次更新的時間戳
`orderId`|integer|`469961015902208000`|訂單ID
`clientOrderId`|string|`213443`|用戶定義的訂單ID
`symbol`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`8200`|訂單價格
`leverage`|float|`4`|訂單杠杆
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|訂單已執行數量
`avgPrice`|float|`4754.24`|平均交易價格
`marginLocked`|float|`200`|該訂單鎖定的保證金。這包括實際需要的保證金外加開倉和平倉所需的費用。
`orderType`|string|`YES`|訂單類型（`LIMIT`和`STOP`）
`priceType`|string|`INPUT`|價格類型（`INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET`）
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`status`|string|`NEW`|訂單狀態（`NEW`、`PARTIALLY_FILLED`、`FILLED`、`CANCELED`、`REJECTED`）
`timeInForce`|string|`GTC`|時效單（Time in Force)類型(`GTC`、`FOK`、`IOC`、`LIMIT_MAKER`)
`fees`|||訂單的手續費

在`fees`資訊組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價類型
`fee`|float|`0`|實際手續費

#### **Example:**

```js
{
'time': '1570760254539',
'updateTime': '0',
'orderId': '469965509788581888',
'clientOrderId': '1570760253946',
'symbol': 'BTC-PERP-REV',
'price': '8502.34',
'leverage': '20',
'origQty': '222',
'executedQty': '0',
'avgPrice': '0',
'marginLocked': '0.00130552',
'orderType': 'LIMIT',
'side': 'BUY_OPEN',
'fees': [],
'timeInForce': 'GTC',
'status': 'NEW',
'priceType': 'INPUT'
}
```

### `myTrades`
返回帳戶的成交歷史，這個API端點需要請求簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/contract/v1/myTrades
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`NO`|| 合約名稱。如果沒有發送，所有合約的成交歷史都會被返回。
`limit`|integer|`NO`|`20`|返回限制(最大值為1000)
`side`|string|`NO`||訂單方向
`fromId`|integer|`NO`||從TradeId開始（用來查詢成交訂單）
`toId`|integer|`NO`||到TradeId結束（用來查詢成交訂單）

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單生成是的時間戳
`tradeId`|long|`49366`|成交ID
`orderId`|long|`630491436`|訂單ID
`matchOrderId`|long|`630491432`| 成交對手訂單ID
`symbolId`|string|`BTC-PERP-REV`|合約名稱
`price`|float|`4765.29`|成交價格
`quantity`|float|`1.01`|成交數量
`feeTokenId`|string|`USDT`|手續費類型（Token名稱）
`fee`|||實際手續費
`side`|string|`BUY`|訂單方向（`BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`）
`orderType`|string|`LIMIT`|訂單類型（`LIMIT`、`MARKET`)

#### **Example:**

```js
[
{
'time': '1570760582848',
'tradeId': '469968263995080704',
'orderId': '469968263793737728',
"matchOrderId": 436002617267469062,
'accountId': '456552319339779840',
'symbolId': 'BTC-PERP-REV',
'price': '8531.17',
'quantity': '100',
'feeTokenId': 'TBTC',
'fee': '0.00000586',
'type': 'LIMIT',
'side': 'BUY_OPEN'
},...
]
```

### `positions`

返回現在的倉位資訊，這個API需要請求簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/contract/v1/positions
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||合約名稱。如果沒有發送，所有的合約倉位資訊都會被返回。
`side`|string|`NO`|| 倉位方向，`LONG`（多倉）或者`SHORT`（空倉）。如果沒有發送，兩個方向的倉位資訊都會被返回。

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-PERP-REV`|合約名稱
`side`|string|`LONG`|倉位方向（`LONG`、`SHORT`）
`avgPrice`|float|`100`|平均開倉價格
`position`|float|`20`|開倉數量（張）
`available`|float|`15`|可平倉數量（張）
`leverage`|float|`5`|倉位現在杠杆
`lastPrice`|float|`100`|合約最新市場成交價
`positionValue`|float|`2000`|倉位價值
`flp`|float|`80`|強制平倉價格
`margin`|float|`20`|倉位保證金
`marginRate`|float|`0.2`|當前倉位的保證金率
`unrealizedPnL`|float|`0.0`|當前倉位的未實現盈虧
`profitRate`|float|`0.0000333`|當前倉位的盈利率
`realizedPnL`|float|`6.8`|當前 **合約** 的已實現盈虧

#### **Example:**
```js
[
{
'symbol': 'BTC-PERP-REV',
'side': 'LONG',
'avgPrice': '8183.11',
'position': '1100',
'available': '1100',
'leverage': '6',
'lastPrice': '8572.53',
'positionValue': '0.12833346',
'flp': '7523.65',
'margin': '0.01251335',
'marginRate': '0.14',
'unrealizedPnL': '0.00608975',
'profitRate': '0.0000333',
'realizedPnL': '-0.00006721'
},...
]
```

### `account`

返回合約帳戶餘額，這個端點需要請求簽名。

#### **Request Weight:**
1

#### **Request Url:**
```bash
GET /openapi/contract/v1/account
```

#### **Parameters:**
None

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`total`|float|`131.06671401`|總餘額
`availableMargin`|float|`131.0545541`|可用保證金
`positionMargin`|float|`0.01215991`|倉位保證金
`orderMargin`|float|`0`|委託保證金（下單鎖定）

#### **Example:**

```js
{
"TBTC": {
"total":"131.06671401",
"availableMargin":"131.0545541",
"positionMargin":"0.01215991",
"orderMargin":"0"
},...
}
```

### `modifyMargin`

更改某個合約倉位的保證金，這個端點需要請求簽名。

#### **Request Weight:**
1

#### **Request Url:**
```bash
POST /openapi/contract/v1/modifyMargin
```

#### **Parameters:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`YES`||合約名稱。
`side`|string|`YES`||倉位方向，`LONG`（多倉）或者`SHORT`（空倉）。
`amount`|float|`YES`||增加（正值）或者減少（負值）保證金的數量。請注意這個數量指的是合約標的定價資產（即合約結算的標的）。

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC-PERP-REV`|合約名稱
`margin`|float|`12.3`|更新後的倉位保證金
`timestamp`|long|`1541161088303`|更新時間戳

#### **Example:**
```js
{
'symbol':'BTC-PERP-REV',
'margin': 15,
'timestamp': 1541161088303
}
```


### `withdraw`

按提幣地址進行提幣操作。

#### **Request Weight:**
1

#### **Request Url:**
```bash
POST /openapi/account/v1/withdraw
```

#### **Parameters:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`clientOrderId`|string|`YES`||客戶端發起提幣請求ID。
`address`|string|`YES`||提幣地址。
`addressExt`|string|`NO`||EOS tag。
`tokenId`|string|`YES`||tokenId。
`withdrawQuantity`|string|`YES`||提幣數量。
`isQuick`|boolean|`YES`||true 加速提幣 false 正常提幣 礦工費不同

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`success`|boolean|true|
`needBrokerAudit`|boolean|true|是否需要券商審核
`orderId`|long|`423885103582776064`|提幣成功訂單id

#### **Example:**
```js
{
"status": 0,
"data": {
"success": true,
"needBrokerAudit": false, // 是否需要券商審核
"orderId": "423885103582776064" // 提幣成功訂單id
}
}
```


### `withdrawDetail`

按提幣地址進行提幣操作。

#### **Request Weight:**
1

#### **Request Url:**
```bash
POST /openapi/account/v1/withdraw
```

#### **Parameters:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`clientOrderId`|string|`YES`||客戶端發起提幣請求ID。
`address`|string|`YES`||提幣地址。
`addressExt`|string|`NO`||EOS tag。
`tokenId`|string|`YES`||tokenId。
`withdrawQuantity`|string|`YES`||提幣數量。
`isQuick`|boolean|`YES`||true 加速提幣 false 正常提幣 礦工費不同

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`success`|boolean|true|
`needBrokerAudit`|boolean|true|是否需要券商審核
`orderId`|long|`423885103582776064`|提幣成功訂單id

#### **Example:**
```js
{
"status": 0,
"data": {
"success": true,
"needBrokerAudit": false, // 是否需要券商審核
"orderId": "423885103582776064" // 提幣成功訂單id
}
}
```

<!-- ### `transfer` **(PENDING)**

This endpoint is used to transfer funds across different accounts. This endpoint requires
you to be signed.

#### **Request Weight:**
1

#### **Request Url:**
```bash
POST /openapi/v1/transfer
```

#### **Parameters:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`from`|string|`YES`||Transfer from which account.
`to`|string|`YES`||Transfer to which account.
`currency`|string|`YES`||The intended currency to transfer. (`USDT`, `BTC`, etc.)
`amount`|float|`YES`||Amount of currency to transfer.

Currently supports transferring assets across `wallet`, `option`, and `contract` accounts.

#### **Response:**

A confirmation message will be returned.

#### **Example:**

```js
{
'message': 'success',
'timestamp': 1541161088303
}
``` -->

### 關鍵參數解釋說明:

#### `side`

交易的方向

`BUY_OPEN`: 開多倉

`SELL_CLOSE`: 平多倉

`SELL_OPEN`: 開空倉

`BUY_CLOSE`: 平空倉

#### `priceType`

價格類型

`INPUT`: 系統將會用你輸入的價格來撮合訂單。

`OPPONENT`: 訂單會以對手盤最優價格撮合。

假設你開多10張合約，盤口最佳買價為10最佳賣價為11，你將會下10張價格為11的合約訂單。如果盤口數量不足成交10張，剩下的將留在盤口。

`QUEUE`: 訂單會以相同方向的最優價格撮合。

假設你開多10張合約，盤口最佳買價為10最佳賣價為11，你將會下10張價格為10的合約訂單。假設盤口原來有5張10的買單，加上你下的單現在一共有15張10的買單。

`OVER`: 訂單會以對手盤的最優價格 + 超價（浮動）撮合

假設你開多10張合約，盤口最佳買價為10最佳賣價為11（超價現在為3），你將會下10張（11+3=）14的買單。如果盤口數量不足成交10張，剩下的將留在盤口。

`MARKET`: 訂單會以 最新成交價 * (1 ± 5%) 撮合

假設你開多10張合約，最新成交價為10，你將會下10張（10*1.05=)10.5的買單。

#### `timeInForce`

時效單類型。

`GTC`: 一直有效直到撤銷。訂單會一直有效除非撤銷。

`IOC`: 馬上成交或者撤銷。訂單會在一個最佳可成交價執行儘量多的交易量, 此訂單可能被部份執行,剩餘的部份將會自動撤銷。

`FOK`: 全部成交或者撤銷。訂單要麼在一個最佳可成交價上全部成交，要麼就會直接撤銷。

`LIMIT_MAKER`: 如果訂單會馬上成交，訂單會被撤銷。

#### `orderType`

訂單類型

`LIMIT`: 訂單會以一個給定（或者更好）的價格成交

`STOP`: 一旦價格到達`triggerPrice`（觸發價），訂單會被觸發

