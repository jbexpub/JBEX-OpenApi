# 通用基本資訊

## 端點資訊

名稱 | 基本端點
------------ | ------------
rest-api | **[https://api.jbex.com](https://api.jbex.com)**
web-socket-streams | **[wss://wsapi.jbex.com](wss://wsapi.jbex.com)**
user-data-stream | **[wss://wsapi.jbex.com](wss://wsapi.jbex.com)*

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

## 限制

* 在 `/openapi/v1/brokerInfo`的`rateLimits` array裏存在當前broker的`REQUEST_WEIGHT`和`ORDER`頻率限制。
* 如果任一頻率限額被超過，`429` 會被返回。
* 每條線路有一個`weight`特性，這個決定了這個請求佔用多少容量（比如`weight`=2說明這個請求佔用兩個請求的量）。返回數據多的端點或者在多個symbol執行任務的端點可能有更高的`weight`。
* 當`429`被返回後，你有義務停止發送請求。
* **多次反復違反頻率限制和/或者沒有在收到429後停止發送請求的用戶將會被收到封禁IP（錯誤碼418）**
* IP封禁會被跟蹤和 **調整封禁時長**（對於反復違反規定的用戶，時間從 **2分鐘到3天不等**）

## 端點安全類型

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
