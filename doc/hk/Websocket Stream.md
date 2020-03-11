# Web Socket推送

## 基本資訊

* 行情基礎端點請參見[這裏](endpoint.md)
* 直接訪問時URL格式為 **/openapi/quote/ws/v1**

| 名稱 | 值 |
| :--- | :---- |
| topic | realtimes, trade, kline_$interval, depth|
| event | sub, cancel, cancel_all|
| interval | 1m, 5m, 15m, 30m, 1h, 2h, 6h, 12h, 1d, 1w, 1M|

**請求訂閱數據樣例:**

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "$topic",
"event": "sub",
// 可調整的參數
"params": {
// kline返回上限是2000，默認為1
"limit": "$limit",
// 返回的數據是否是壓縮過的，默認為false
"binary": "false"
}
}
```

| 名稱 | 解釋 |
| :--- | :---- |
|limit|規定返回結果的數量|
|binary|規定返回的數據是否是壓縮過的。**默認**值是**false**。|

## 心跳

每隔一段時間，客戶端需要發送ping幀，服務端會回復pong幀，否則服務端會在5分鐘內主動斷開鏈接。

* 請求
```javascript
{
"ping": 1535975085052
}
```

* 返回
```javascript
{
"pong": 1535975085052
}
```

## 逐筆交易

逐筆交易推送每一筆成交的資訊。成交，或者說交易的定義是僅有一個吃單者與一個掛單者相互交易。

在成功連接到伺服器後，伺服器首先會推送一條最近的60條成交。在這條推送之後，每條推送都是即時的成交。

變數“v”可以理解成一個交易ID。這個變數是全局遞增的並且獨特的。例如：假設過去5秒有3筆交易發生，分別是ETHUSDT、BTCUSDT、JBTBTC。它們的“v”會為連續的值（112，113，114）。

**請求訂閱數據樣例:**

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "trade",
"event": "sub",
"params": {
"binary": false // Whether data returned is in binary format
}
}
```

**返回:**

```javascript
{
"symbol": "BTCUSDT",
"topic": "trade",
"data": [{
"v": "426635153180475392", // 參見解釋
"t": 1565594873508, //時間戳
"p": "11369", // 價格
"q": "0.01", // 數量
"m": false // true = 買, false = 賣
}, {
"v": "426635153373413376",
"t": 1565594873531,
"p": "11369",
"q": "0.0012",
"m": false
}],
"f": false // 是不是第一個返回
}
```

## Symbol的Ticker

按Symbol逐秒刷新的24小時完整ticker資訊

**請求訂閱數據樣例:**

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "realtimes",
"event": "sub",
"params": {
"binary": false
}
}
```

**返回:**

```javascript
{
"symbol": "ETHUSDT",
"topic": "realtimes",
"data": [{
"t": "1565592599015", //時間戳
"s": "ETHUSDT", //symbol
"c": "212.63", //收盤價
"h": "216.96", //最高價
"l": "206.78", //最低價
"o": "210.23", //開盤價
"v": "73013.575", //交易量
"qv": "15726612.498168", //交易額
}],
"f": false // 是否為第一個返回
}
```

## K線/蠟燭圖

K線stream逐秒推送所請求的K線種類(最新一根K線)的更新

**K線/蠟燭圖間隔:**

訂閱Kline需要提供間隔參數，最短為分鐘線，最長為月線。支持以下間隔:

m -> 分鐘; h -> 小時; d -> 天; w -> 周; M -> 月

* 1m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 12h
* 1d
* 1w
* 1M

**請求訂閱數據樣例:**

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "kline_"+$間隔,
"event": "sub",
"params": {
"binary": false
}
}
```

**返回:**

```javascript
{
"symbol": "BTCUSDT",
"topic": "kline",
"params": {"klineType": "15m"},
"data": [{
"t": 1565595900000, //k線開始時間
"s": "BTCUSDT", // symbol
"c": "11436.14", //收盤價
"h": "11437", //最高價
"l": "11381.89", //最低價
"o": "11381.89", //開盤價
"v": "16.3306" //交易量
}],
"f": true // 是否為第一個返回
}
```

## 訂單簿深度資訊

Symbol的深度資訊。

這裏是訂單簿快照推送的詳細資訊：
* 訂單簿快照頻率：每300ms, 如果book變了的話。
* 訂單簿快照頻率深度：bids 和 asks各300
* 訂單簿版本變更觸發事件：
* 訂單進入訂單簿
* 訂單離開訂單簿
* 訂單數量變更
* 訂單已完成

### 有限檔深度資訊

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "depth",
"event": "sub",
"params": {
"binary": false
}
}
```

**返回:**

```javascript
{
"symbol": "BTCUSDT",
"topic": "depth",
"data": [{
"s": "BTCUSDT", //Symbol
"t": 1565600357643, //時間戳
"v": "112801745_18", //見上面解釋
"b": [ //Bids
["11371.49", "0.0014"], //[價格, 數量]
["11371.12", "0.2"],
["11369.97", "0.3523"],
["11369.96", "0.5"],
["11369.95", "0.0934"],
["11369.94", "1.6809"],
["11369.6", "0.0047"],
["11369.17", "0.3"],
["11369.16", "0.2"],
["11369.04", "1.3203"],
"a": [//Asks
["11375.41", "0.0053"], //[價格, 數量]
["11375.42", "0.0043"],
["11375.48", "0.0052"],
["11375.58", "0.0541"],
["11375.7", "0.0386"],
["11375.71", "2"],
["11377", "2.0691"],
["11377.01", "0.0167"],
["11377.12", "1.5"],
["11377.61", "0.3"]
]
}],
"f": true//是否為第一個返回
}
```

### 增量深度資訊

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "diffDepth",
"event": "sub",
"params": {
"binary": false
}
}
```

每秒推送訂單簿的變化部分（如果有）。

在增量深度資訊中，數量不一定等於對應價格的數量。如果數量=0，這說明在上一條推送中的這個價格已經沒有了。如果數量>0，這時的數量為更新後的這個價格所對應的數量

假設我們收到的返回數據中有這樣一條：

```javascript
["0.00181860", "155.92000000"]// 價格，數量
```

如果下一條返回數據中有：
```javascript
["0.00181860", "12.3"]
```
這說明這個價格對應的數量有變更，已經更新變更的數量

如果下一條返回數據中有：
```javascript
["0.00181860", "0"]
```
這說明這個價格對應的數量已經消失，將會在客戶端中刪除。

**Payload:**

```javascript
{
"symbol": "BTCUSDT",
"topic": "diffDepth",
"data": [{
"e": 0,
"t": 1565687625534,
"v": "115277986_18",
"b": [
["11316.78", "0.078"],
["11313.16", "0.0052"],
["11312.12", "0"],
["11309.75", "0.0067"],
["11309.58", "0"],
["11306.14", "0.0073"]
],
"a": [
["11318.96", "0.0041"],
["11318.99", "0.0017"],
["11319.12", "0.0017"],
["11319.22", "0.4516"],
["11319.23", "0.0934"],
["11319.24", "3.0665"]
]
}],
"f": false //是否為第一個返回值
}
```

## 指數數據

期權和期貨指數數據。

**請求訂閱數據樣例:**

```javascript
{
"symbol": "$symbol0, $symbol1",
"topic": "index",
"event": "sub",
}
```

**返回:**

```javascript
{
"symbol": "HTUSDT",
"topic": "index",
"data": [{
"symbol": "HTUSDT", //symbol
"index": "5.0941", //當前指數價
"edp": "5.08799333", //預計交割價，見期權rest文檔
"formula": "(5.0941[HUOBI])/1" //來源
}],
"f": true //是否為第一個返回
}
```

## 錯誤處理

**錯誤碼:**

```java
INVALID_REQUEST("-10000", "Invalid request!")
JSON_FORMAT_ERROR("-10001", "Invalid JSON!")
INVALID_EVENT("-10002", "Invalid event")
REQUIRED_EVENT("-10003", "Event required!")
INVALID_TOPIC("-10004", "Invalid topic!")
REQUIRED_TOPIC("-10005", "Topic required!")
PARAM_EMPTY("-10007", "Params required!")
PERIOD_EMPTY("-10008", "Period required!")
PERIOD_ERROR("-10009", "Invalid period!")
SYMBOLS_ERROR("-100010", "Invalid Symbols!")
```

