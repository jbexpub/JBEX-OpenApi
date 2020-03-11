# 期權交易

Broker Open API的地址請見[這裏](endpoint.md)

## 公共接口

### `brokerInfo`

獲取當前broker的交易規則和symbol的資訊（精度單位等資訊）

#### **Request Weight:**

0

#### **Request Url:**
```bash
GET /openapi/v1/brokerInfo
```

#### **Parameters:**

None

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`timezone`|string|`UTC`|伺服器所在時區
`serverTime`|long|`1554887652929`|當前伺服器時間（Unix Timestamp格式，ms毫秒級)


在`symbols`對應的資訊組裏，顯示的是幣幣交易的symbol的資訊（精度等），與期權交易並無關聯，可以忽略。

在 `options`對應的資訊組裏，所有當前正在交易的期權資訊將會被返回：

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC0308CS3900`|期權名稱
`status`|string|`TRADING`|期權當前狀態
`baseAsset`|string|`BTC0308CS3900`|期權的名稱
`baseAssetPrecision`|float|`0.001`|期權交易張數精度
`quoteAsset`|string|`BUSDT`|計價的貨幣
`quoteAssetPrecision`|float|`0.01`|期權交易價格的精度
`icebergAllowed`|string|`false`|是否支持“冰山訂單”

在`options`裏面的`filters`對應的資訊組裏：

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`filterType`|string|`PRICE_FILTER`|Filter類型
`minPrice`|float|`0.001`|期權最小交易價格
`maxPrice`|float|`100000.00000000`|
`tickSize`|float|`0.001`|期權交易價格精度
`minQty`|float|`0.01`|期權最小交易張數
`maxQty`|float|`100000.00000000`|
`stepSize`|float|`0.001`|期權交易張數精度
`minNotional`|float|`1`|訂單金額精度 (數量 * 價格)

#### **Example:**
```js
{
'timezone': 'UTC',
'serverTime': '1555048558151',
'brokerFilters': [],
'symbols': [{...}],
'options': [
{
'filters': [
{
'minPrice': '0.01',
'maxPrice': '100000.00000000',
'tickSize': '0.01',
'filterType': 'PRICE_FILTER'
},
{
'minQty': '0.01',
'maxQty': '100000.00000000',
'stepSize': '0.001',
'filterType': 'LOT_SIZE'
},
{
'minNotional': '1',
'filterType': 'MIN_NOTIONAL'
}
],
'exchangeId': '301',
'symbol': 'BTC0412PS5100',
'status': 'TRADING',
'baseAsset': 'BTC0412PS5100',
'baseAssetPrecision': '0.001',
'quoteAsset': 'USDT',
'quotePrecision': '0.01',
'icebergAllowed': False
},...
]
} 
```

### `getOptions`

獲取所有正在交易和已經交割的期權資訊。如果需要獲取歷史期權資訊，需要將`expired`設置成`true`

#### **Request Weight:**

1

#### **Request URL:**
```
GET /openapi/v1/getOptions
```

#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -----
`expired`|string|`NO`|`false`|設置為`true`來展示歷史期權，可以用來獲取歷史期權資訊。

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC0412CS5400`|期權名稱。命名規則為：`標的資產-交割時間-期權類型（看漲CS/看跌PS）-行權價`
`strike`|float|`5400.0`|期權的行權價
`created`|long|`1554710400000`|期權開始交易時的Unix Timestamp（毫秒ms)
`expiration`|long|`1555055400000`| 期權結束交易時的Unix Timestamp（毫秒ms)
`optionType`|integer|`1`|期權類型，`1`=看漲期權，`0`= 看跌期權
`maxPayoff`|float|`500`|期權的最大收益
`underlying`|string|`BTCBUSDT`|期權的標的的指數價格名稱
`settlement`|string|`weekly`|結算區間。`weekly`=每週。


#### **Example:**
```js
[
{'symbol': 'BTC0412PS5100',
'strike': '5100.0',
'created': '1554710400000',
'expiration': '1555055400000',
'optionType': 0,
'maxPayOff': '500.0',
'underlying': 'BTCUSDT',
'settlement': 'weekly'
},...
]
```

### `index`
獲取當前指數價格和EDP（預估交割價格）。這個端點不用發送任何參數。

#### **Request Weight:**

0

#### **Request URL:**
```
GET /quote/v1/option/index
```

#### **Parameters:**
None

#### **Response：**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`index`|float|`3652.81`|當前指數價格。
`edp`|float|`3652.81`|預估交割價格（過去10分鐘指數價格的平均值）

#### **Example:**
```js
{
'BTCUSDT':{
'index':3795.77,
'edp': 3652.81
},
...
}
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
GET /openapi/quote/v1/option/depth
```

#### **Parameters:**

名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -----
`symbol`|string|`YES`||用來獲取訂單簿的期權名稱。使用`getOptions`來獲取期權名稱。
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
'time': 1555049455783,
'bids': [
['78.82', '0.526'],//[價格，數量]
['77.24', '1.22'],
['76.65', '1.043'],
['76.58', '1.34'],
['75.67', '1.52'],
['75.12', '0.635'],
['75.02', '0.72'],
['75.01', '0.672'],
['73.73', '1.282'],
['73.58', '1.116'],
['73.45', '0.471'],
['73.44', '0.483'],
['72.32', '0.383'],
['72.26', '1.283'],
['72.11', '0.703'],
['70.61', '0.454']],
'asks': [
['122.96', '0.381'],//[價格，數量]
['144.46', '1'],
['155.55', '0.065'],
['160.16', '0.052'],
['200', '0.775'],
['249', '0.17'],
['250', '1'],
['300', '1'],
['400', '1'],
['499', '1']]
}

```

### `trades`

獲取某個期權最近成交訂單的資訊。

#### **Request Weight:**

1

#### **Request URL:**
```
GET /openapi/quote/v1/option/trades
```
#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | -------
`symbol`|string|`YES`||期權名稱
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
'price': '1.21',
'time': 1555034474064,
'qty': '0.725',
'isBuyerMaker': False
},...
]
```

### `klines`

獲取某個期權的K線資訊（高，低，開，收，交易量...)

#### **Request Weight:**

1

#### **Request URL:**
```
GET /openapi/quote/v1/option/klines
```

#### **Parameters：**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | ------
`symbol`|string|`YES`||期權名稱
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
`''`|float|`148976.11427815`|期權交易金額
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
'148976.11427815', // 期權交易金額
1499644799999, // 收盤時間
'2434.19055334', // 交易數量（張數）
308, // 已成交數量（張數）
'1756.87402397', // 買方購買金額
'28.46694368' // 買方購買數量（張數）
],...
]
```

## 私有接口

### `order`

下一個做多（buy，即買入）或者做空（sell，即賣出）期權的訂單。這個期權端點需要你的簽名

#### **Request Weight:**

1

#### **Request URL:**
```bash
POST /openapi/option/v1/order
```

#### **Parameters：**

名稱|類型|是否強制|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`YES`|期權名稱
`clientOrderId`|string/long|`NO`|訂單的ID。可自己定義，如果沒有發送，將會自動生成。
`side`|string|`YES`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`type`|string|`YES`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`timeInForce`|string|`NO`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`price`|float|`NO` Required for limit orders|訂單的價格
`quantity`|float|`YES`|訂單購買的數量（張數）

你可以從`brokerInfo`中獲取期權價格，數量的配置資訊。

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單創建時的時間戳，毫秒（ms）
`updateTime`|long|`1551062936784`|上次訂單更新時間，毫秒（ms)
`orderId`|integer|`891`|訂單ID（系統生成）
`clientOrderId`|integer|`213443`|訂單ID（自己發送的）
`symbol`|string|`BTC0412CS4200`|期權名稱
`price`|float|`4765.29`|訂單價格
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|已經成交訂單數量
`avgPrice`|float|`4754.24`|訂單已經成交的平均價格
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`status`|string|`NEW`|訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`timeInForce`|string|`GTC`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`fees`|||訂單產生的手續費

在`fees`裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價單位
`fee`|float|`0`|實際費用值

#### **Example:**
```js
{
'time':1541161088303,
'updateTime': 1541161088303,
'orderId': 28,
'clientOrderId': 213443,
'symbol': 'BTC0412CS4200',
'price': 102.32,
'origQty': 21.3,
'executedQty': 10.2,
'avgPrice': 3121.13
'type': 'LIMIT',
'side': 'SELL',
'status': 'NEW',
'timeInForce': 'GTC',
'fees':[]
}
```

### `cancel`

取消一個訂單，用`orderId` 或者 `clientOrderId`來取消。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
DELETE /openapi/option/v1/order/cancel
```

#### **Parameter:**

名稱|類型|是否強制|描述
------------ | ------------ | ------------ | ------------
`orderId`|integer|`NO`|系統自動生成的訂單ID。
`clientOrderId`|string/long|`NO`|自己傳送的訂單ID。

**必須**傳送以上兩個參數的其中一個。

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單創建時的時間戳，毫秒（ms）
`updateTime`|long|`1551062936784`|上次訂單更新時間，毫秒（ms)
`orderId`|integer|`891`|訂單ID（系統生成）
`clientOrderId`|integer|`213443`|訂單ID（自己發送的）
`symbol`|string|`BTC0412CS4200`|期權名稱
`price`|float|`4765.29`|訂單價格
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|已經成交訂單數量
`avgPrice`|float|`4754.24`|訂單已經成交的平均價格
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`status`|string|`NEW`|訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`timeInForce`|string|`GTC`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`fees`|||訂單產生的手續費

在`fees`裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價單位
`fee`|float|`0`|實際費用值

#### **Example:**

```js
{
'time':1541161088303,
'updateTime': 1541161088303,
'orderId': 713637304,
'clientOrderId': 213443,
'symbol': 'BTC0412CS4200',
'price': 102.32,
'origQty': 21.3,
'executedQty': 10.2,
'avgPrice': 3121.13
'type': 'LIMIT',
'side': 'SELL',
'status': 'CANCELED', //cancel請求的訂單狀態會一直為`CANCELED`
'timeInForce': 'GTC',
'fees': []
}
```

#### `openOrders`

獲取你當前未成交的訂單。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/openOrders
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`||期權名稱，如果沒有發送默認返回所有期權訂單。
`orderId`|integer|`NO`||訂單ID。
`side`|string|`NO`||訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`type`|string|`NO`||訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`limit`|integer|`NO`|`20`|返回值的數量。

如果發送了`orderId`，將會返回小於`orderId`的所有訂單。若沒有，將會返回最新的訂單。

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單創建時的時間戳，毫秒（ms）
`updateTime`|long|`1551062936784`|上次訂單更新時間，毫秒（ms)
`orderId`|integer|`891`|訂單ID（系統生成）
`clientOrderId`|integer|`213443`|訂單ID（自己發送的）
`symbol`|string|`BTC0412CS4200`|期權名稱
`price`|float|`4765.29`|訂單價格
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|已經成交訂單數量
`avgPrice`|float|`4754.24`|訂單已經成交的平均價格
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`status`|string|`NEW`|訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`timeInForce`|string|`GTC`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`fees`|||訂單產生的手續費

在`fees`裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價單位
`fee`|float|`0`|實際費用值

#### **Example:**

```js
[
{
'time': '1554948456641',
'updateTime': '0',
'orderId': '337326535438529024',
'clientOrderId': '19524737',
'symbol': 'BTC0412CS4200',
'price': '1.98',
'origQty': '1',
'executedQty': '0',
'avgPrice': '0',
'type': 'LIMIT',
'side': 'BUY',
'status': 'NEW',
'timeInForce': 'GTC',
'fees': []
},...

]
```

### `positions`

獲取當前倉位資訊。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/positions
```

#### **Parameters:**

名稱|類型|是否強制|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`NO`|期權名稱，如果沒有發送默認返回所有期權的倉位。

#### **Response:**
對於每個`symbol`（期權名稱），這個端點將會返回以下資訊。

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC0405PS3850`|期權名稱
`position`|float|`-10.760`|該期權的持倉量（張數）。可以為正（做多）也可以為負（做空）。
`margin`|float|`5380`|當前倉位的總保證金量
`settlementTime`|integer|`1555056000000`|該期權的交割時間戳，毫秒（ms）
`strikePrice`|float|`4200`|該期權的行權價
`price`|float|`500.00`|當前期權價格
`availablePosition`|float|`10.76`|可平倉數量（張數）
`averagePrice`|float|`9693.502194671`|持倉均價（持有倉位的成交金額/持倉量）
`changed`|float|`-4018.21`|持倉盈虧。**做多：** （最新價-持倉均價）\* 持倉量 **做空：**（最新價-持倉均價）\* 持倉量 \*(-1)
`changedRate`|float|`1.02`|持倉盈虧百分比。**做多：** 持倉盈虧/ （持倉均價 \* 持倉量） **做空：** 持倉盈虧/（保證金 - 持倉均價 \* 持倉量）
`index`|float|`5012.28666667`|當前標的資產指數值

#### **Example:**

```js
[
{
'symbol': 'BTC0412CS4200',
'position': '-10.760',
'margin': '5380',
'settlementTime': '1555056000000',
'strikePrice': '4200',
'price': '500.00',
'availablePosition': '10.76',
'averagePrice': '126.56',
'changedRate': '-100.00',
'changed': '-4018.21',
'index': '5012.28666667'
},...
]
```

### `historyOrders`

獲取歷史訂單資訊（部分成交的、全部成交的、取消的）。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/historyOrders
```

#### **Parameters:**
Parameter|type|required|default|description
------------ | ------------ | ------------ | ------------ | ---------
`symbol`|string|`NO`||期權名稱，如果沒有發送將默認返回所有期權的訂單。
`side`|string|`NO`||訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`type`|string|`NO`||訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`orderStatus`|string|`NO`||訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`limit`|integer|`NO`|`20`|返回值的數量

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單創建時的時間戳，毫秒（ms）
`updateTime`|long|`1551062936784`|上次訂單更新時間，毫秒（ms)
`orderId`|integer|`891`|訂單ID（系統生成）
`clientOrderId`|integer|`213443`|訂單ID（自己發送的）
`symbol`|string|`BTC0412CS4200`|期權名稱
`price`|float|`4765.29`|訂單價格
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|已經成交訂單數量
`avgPrice`|float|`4754.24`|訂單已經成交的平均價格
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`status`|string|`NEW`|訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`timeInForce`|string|`GTC`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`fees`|||訂單產生的手續費

在`fees`裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價單位
`fee`|float|`0`|實際費用值

#### **Example:**
```js
{
[
{
'time':1541161088303,
'updateTime': 1541161088303,
'orderId': 28,
'clientOrderId': 213443,
'symbol': 'BTC0412CS4200',
'price': 102.32,
'origQty': 21.3,
'executedQty': 10.2,
'avgPrice': 3121.13
'type': 'LIMIT',
'side': 'SELL',
'status': 'NEW',
'timeInForce': 'GTC',
'fees':[]
},...
]
}
```

### `getOrder`

獲取某個訂單的詳細資訊

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/getOrder
```

#### **Parameters:**
名稱|類型|是否強制|默認|描述
------------ | ------------ | ------------ | ------------ | --------
`orderId`|integer|`NO`||訂單ID
`clientOrderId`|string|`NO`||用戶定義的訂單ID

**注意：**` orderId` 或者 `clientOrderId` **必須發送其中之一**

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1551062936784`|訂單創建時的時間戳，毫秒（ms）
`updateTime`|long|`1551062936784`|上次訂單更新時間，毫秒（ms)
`orderId`|integer|`891`|訂單ID（系統生成）
`clientOrderId`|integer|`213443`|訂單ID（自己發送的）
`symbol`|string|`BTC0412CS4200`|期權名稱
`price`|float|`4765.29`|訂單價格
`origQty`|float|`1.01`|訂單數量
`executedQty`|float|`1.01`|已經成交訂單數量
`avgPrice`|float|`4754.24`|訂單已經成交的平均價格
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`status`|string|`NEW`|訂單狀態。可能出現的值為：`NEW`(新訂單，無成交)、`PARTIALLY_FILLED`（部分成交）、`FILLED`（全部成交）、`CANCELED`（已取消）和`REJECTED`（訂單被拒絕）.
`timeInForce`|string|`GTC`|訂單時間指令（Time in Force）。可能出現的值為：`GTC`（Good Till Canceled，一直有效），`FOK`（Fill or Kill，全部成交或者取消），`IOC`（Immediate or Cancel，立即成交或者取消）.
`fees`|||訂單產生的手續費

在`fees`裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`feeToken`|string|`USDT`|手續費計價單位
`fee`|float|`0`|實際費用值

#### **Example:**

```js
{
'time':1541161088303,
'updateTime': 1541161088303,
'orderId': 28,
'clientOrderId': 213443,
'symbol': 'BTC0412CS4200',
'price': 102.32,
'origQty': 21.3,
'executedQty': 10.2,
'avgPrice': 3121.13
'type': 'LIMIT',
'side': 'SELL',
'status': 'NEW',
'timeInForce': 'GTC',
'fees':[]
}
```

### `myTrades`

獲取當前帳戶的成交訂單記錄。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/myTrades
```

#### **Parameters:**
Parameter|type|required|default|description
------------ | ------------ | ------------ | ------------ | --------
`symbol`|string|`NO`|| 期權名稱，如果沒有發送將默認返回所有期權的訂單。
`limit`|integer|`NO`|`20`|返回值的數量 (最大值為1000)
`side`|string|`NO`||訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`fromId`|integer|`NO`||大於這個值的tradeId的訂單
`toId`|integer|`NO`||小於這個值的tradeId的訂單

#### **Response:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`time`|long|`1503439494351`|訂單成交時的時間戳，毫秒（ms）
`tradeId`|long|`49366`|成交訂單ID
`orderId`|long|`630491422`| 訂單ID
`matchOrderId`|long|`630491432`| 成交對方訂單ID
`price`|float|`0.055`|訂單價格
`quantity`|float|`23.3`|訂單數量
`feeTokenName`|string|`USDT`|手續費計價單位
`fee`|float|`0.000090000000000000`|手續費用值
`side`|string|`BUY`|訂單方向。可能出現的值只能為：`BUY`（買入做多） 和 `SELL`（賣出做空）
`type`|string|`LIMIT`|訂單類型。可能出現的值只能為:`LIMIT`(限價)和`MARKET`（市價）
`symbol`|string|`BTC0412PS3900`|期權名稱

#### **Example:**
```js
[
{
'time': '1554897921663',
'tradeId': '336902617393292032',
'orderId': '336902617267462912',
"matchOrderId": 336002617267469062,
'price': '99',
'quantity': '11.414',
'feeTokenName': 'BUSDT',
'fee': '0.1129986',
'type': 'LIMIT',
'side': 'BUY',
'symbol': 'BTC0412PS3900'
},...
]
```

### `settlements`

獲取你帳戶裏交割期權的資訊。這個API端點需要你的簽名。

#### **Request Weight:**

1

#### **Request Url:**
```bash
GET /openapi/option/v1/settlements
```
#### **Parameters:**
None

#### **Responses:**

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`symbol`|string|`BTC0412PS3900`|期權名稱
`optionType`|string|`call`|期權類型
`margin`|float|`400`|倉位所需的保證金
`timestamp`|integer|`1517299201923`|交割時間的時間戳，毫秒（ms）。
`strikePrice`|float|`3740`|當前期權的行權價
`settlementPrice`|float|`11008.37`|期權結算價格
`maxPayoff`|float|`400`|期權的最大收益
`averagePrice`|float|`9693.502194671`|持倉均價
`position`|string|`1000`|持倉量（張）
`changed`|float|`20.3`|交割收益
`changedRate`|float|`2.34`|交割收益百分比。 **做多：** 做多交割收益/ (持倉均價 \* 持倉量) **做空：**做空交割收益/（保證金-持倉均價 \* 持倉量）

#### **Examples:**
```js
[
{'symbol': 'BTC0405PS3850',
'optionType': 'put',
'margin': '0',
'timestamp': '1554451200000',
'strikePrice': '3850',
'settlementPrice': '4956.54',
'maxPayOff': '500',
'averagePrice': '119.27',
'position': '0',
'changed': '0',
'changedRate': '0'},...
] 
```

### `account`

獲取當前帳戶餘額資訊。這個API端點需要你的簽名。

#### **Request Weight:**
1

#### **Request Url:**
```bash
GET /openapi/option/v1/account
```

#### **Parameters:**
None

#### **Response:**
名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`totalAsset`|float|`1000.0`|期權帳戶中的全部資產估值（以USDT計算）
`optionAsset`|float|`100.0`|期權帳戶中的期權估值（以USDT計算）
`balances`|float||展示餘額數據

在`balances`數據組裏:

名稱|類型|例子|描述
------------ | ------------ | ------------ | ------------
`tokenName`|string|`USDT`|資產的名稱
`free`|float|`600.0`|可用額
`locked`|float|`100.0`|凍結額（未成交訂單凍結）
`margin`|float|`100.0`|保證金 （做空期權抵押）

#### **Examples:**
```js
{
'totalAsset': '8533.0606762',
'optionAsset': '558.1832',
'balances': [
{
'tokenName': 'USDT',
'free': '0.0',
'locked': '0.0',
'margin': '0.0'
},
{
'tokenName': 'BUSDT',
'free': '7961.9951881',
'locked': '12.8822881',
'margin': '5798.0'
},...
]
}
```

