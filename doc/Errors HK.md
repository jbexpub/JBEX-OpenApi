# 錯誤碼

返回報錯壹般由兩個部分組成：錯誤碼和錯誤資訊。錯誤碼是通用的，但是錯誤資訊會有所不同。如下是壹個報錯JSON Payload示例：

```javascript
{
  "code":-1121,
  "msg":"Invalid symbol."
}
```

## 10xx - 通用服務器和網絡錯誤

### -1000 UNKNOWN

* 處理請求時發生未知錯誤

### -1001 DISCONNECTED

* 內部錯誤; 無法處理您的請求。 請再試壹次.

### -1002 UNAUTHORIZED

* 您無權執行此請求。請求需要發送API Key，我們建議在所有的請求種豆附加API Key 

### -1003 TOO_MANY_REQUESTS

* 排隊的請求過多。  
* 請求權重過多； 請使用websocket進行實時更新。  
* 請求權重過多； 當前限制為每分鐘％s請求權重。 請使用websocket進行實時更新，以避免輪詢API
* 請求權重過多； IP被禁止，直到％s。 請使用websocket進行實時更新，以免被禁。

### -1006 UNEXPECTED_RESP

* 從消息總線收到意外的響應。 執行狀態未知。請向客服求證關於此訂單的詳細狀態和其他資訊。

### -1007 TIMEOUT

* 等待後端服務器響應超時。 發送狀態未知； 執行狀態未知。

### -1014 UNKNOWN_ORDER_COMPOSITION

* 不支持的訂單組合。

### -1015 TOO_MANY_ORDERS

* 新訂單太多。請減少妳的請求頻率
* 新訂單太多； 當前限制為每％s ％s個訂單。

### -1016 SERVICE_SHUTTING_DOWN

* 該服務不可用

### -1020 UNSUPPORTED_OPERATION

* 不支持此操作

### -1021 INVALID_TIMESTAMP

* 此請求的時間戳不在recvWindow之外。
* 此請求的時間戳比服務器時間提前1000毫秒。
* 請查證妳的本地時間和服務器時間

### -1022 INVALID_SIGNATURE

* 此請求的簽名無效。

## 11xx - 請求問題

### -1100 ILLEGAL_CHARS

* 在參數中發現非法字符。 
* 在參數'％s'中發現非法字符； 合法範圍是“％s”。

### -1101 TOO_MANY_PARAMETERS

* 為此端點發送的參數太多。  
* 參數太多； 預期為“％s”並收到了“％s”。 
* 檢測到的參數值重復。

### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED

* 未發送強制性參數，該參數為空/空或格式錯誤。  
* 強制參數'％s'未發送，為空/空或格式錯誤。  
* 必須發送參數'％s'或'％s'，但兩者均為空！

### -1103 UNKNOWN_PARAM

* 發送了未知參數。
* 每條請求需要至少壹個參數{Timestamp}.

### -1104 UNREAD_PARAMETERS

* 並非所有發送的參數都被讀取。
* 並非所有發送的參數都被讀取； 讀取了'％s'參數，但被發送了'％s'。

### -1105 PARAM_EMPTY

* 參數為空。
* 參數'％s'為空。

### -1106 PARAM_NOT_REQUIRED

* 不需要時已發送參數。
* 不需要時發送參數'％s'。

### -1111 BAD_PRECISION

* 精度超過為此資產定義的最大值。

### -1112 NO_DEPTH

* 交易對沒有掛單

### -1114 TIF_NOT_REQUIRED

* 不需要時發送了TimeInForce參數。

### -1115 INVALID_TIF

* 無效 timeInForce.

### -1116 INVALID_ORDER_TYPE

* 無效訂單類型。

### -1117 INVALID_SIDE

* 無效買賣方向。

### -1118 EMPTY_NEW_CL_ORD_ID

* 新的客戶訂單ID為空。

### -1119 EMPTY_ORG_CL_ORD_ID

* 原始客戶訂單ID為空。

### -1120 BAD_INTERVAL

* 無效時間間隔。

### -1121 BAD_SYMBOL

* 無效符號。

### -1125 INVALID_LISTEN_KEY

* 該listenKey不存在。

### -1127 MORE_THAN_XX_HOURS

* 查詢間隔太大。
* 從開始時間到結束時間之間超過％s小時。

### -1128 OPTIONAL_PARAMS_BAD_COMBO

* 可選參數組合無效。

### -1130 INVALID_PARAMETER

* 發送的參數為無效數據。
* 發送參數'％s'的數據無效。

### -1132 ORDER_PRICE_TOO_HIGH

* 訂單價格過高

### -1133 ORDER_PRICE_TOO_SMALL

* 訂單價格過低，請查詢Broker Info資訊

### -1134 ORDER_PRICE_PRECISION_TOO_LONG

* 訂單價格精度過長，請查詢Broker Info資訊

### -1135 ORDER_QUANTITY_TOO_BIG

* 訂單quantity過大

### -1136 ORDER_QUANTITY_TOO_SMALL

* 訂單quantity小於最小值

### -1137 ORDER_QUANTITY_PRECISION_TOO_LONG

* 訂單quantity精度過長

### -1138 ORDER_PRICE_WAVE_EXCEED

* 訂單價格超出允許範圍

### -1139 ORDER_HAS_FILLED

* 訂單已經被執行

### -1140 ORDER_AMOUNT_TOO_SMALL

* 交易金額小於最小值

### -1141 ORDER_DUPLICATED

* clientOrderId重復

### -1142 ORDER_CANCELLED

* 訂單已經被撤銷

### -1143 ORDER_NOT_FOUND_ON_ORDER_BOOK

* 訂單當前不在訂單簿上

### -1144 ORDER_LOCKED

* 訂單已被鎖定

### -1145 ORDER_NOT_SUPPORT_CANCELLATION

* 該訂單類型不支持撤銷

### -1146 ORDER_CREATION_TIMEOUT

* 訂單生成超時

### -1147 ORDER_CANCELLATION_TIMEOUT

* 訂單撤銷超時

### -2010 NEW_ORDER_REJECTED

* NEW_ORDER_REJECTED

### -2011 CANCEL_REJECTED

* CANCEL_REJECTED

### -2013 NO_SUCH_ORDER

* Order不存在

### -2014 BAD_API_KEY_FMT

* API-key格式無效

### -2015 REJECTED_MBX_KEY

* 無效的API密鑰，IP或操作權限。.

### -2016 NO_TRADING_WINDOW

* 找不到該交易對的交易窗口。 嘗試改為24小時自動報價。

## -1010 ERROR_MSG_RECEIVED, -2010 NEW_ORDER_REJECTED, -2011 CANCEL_REJECTED的錯誤資訊

匹配引擎返回錯誤後，將發送此代碼。 以下消息將指示特定的錯誤：

Error message | Description
------------ | ------------
"Unknown order sent." | 找不到訂單（通過`orderId`，`clientOrderId`，`origClientOrderId`）
"Duplicate order sent." | `clOrdId`已經被使用
"Market is closed." | 該交易對不在交易範圍
"Account has insufficient balance for requested action." | 沒有足夠的資金來完成行動
"Market orders are not supported for this symbol." | 交易對上未啟用`MARKET`
"Iceberg orders are not supported for this symbol." | 該交易對未啟用`icebergQty`
"Stop loss orders are not supported for this symbol." | 該交易對未啟用`STOP_LOSS`
"Stop loss limit orders are not supported for this symbol." | 該交易對未啟用`STOP_LOSS_LIMIT`
"Take profit orders are not supported for this symbol." | 該交易對未啟用`TAKE_PROFIT`
"Take profit limit orders are not supported for this symbol." | 該交易對未啟用`TAKE_PROFIT_LIMIT`
"Price* QTY is zero or less." | `price`* `quantity`太小
"IcebergQty exceeds QTY." | `icebergQty` 必須小於訂單數量
"This action disabled is on this account." | 聯系客戶支持；該帳戶已禁用了某些操作。
"Unsupported order combination" | 不允許組合`orderType`, `timeInForce`, `stopPrice`, 和/或 `icebergQty`。
"Order would trigger immediately." | 與最後交易價格相比，訂單的止損價無效。
"Cancel order is invalid. Check origClOrdId and orderId." | 未發送`origClOrdId` 或 `orderId`。
"Order would immediately match and take." | `LIMIT_MAKER`訂單類型將立即匹配並進行交易，而不是純粹的生成訂單。 

## -9xxx 過濾器故障

Error message | Description
------------ | ------------
"Filter failure: PRICE_FILTER" | `價格`過高，過低和/或不遵循交易對的最小價格規則。
"Filter failure: LOT_SIZE" | `數量`太高，太低和/或不遵循該交易對的步長規則。 
"Filter failure: MIN_NOTIONAL" | `價格` * `數量`太低，無法成為該交易對的有效訂單 
"Filter failure: MAX_NUM_ORDERS" | 客戶在交易對上有太多掛單。
"Filter failure: MAX_ALGO_ORDERS" | 客戶在交易對上有太多止損/止盈掛單。
"Filter failure: BROKER_MAX_NUM_ORDERS" | 客戶在當前券商上有太多掛單。
"Filter failure: BROKER_MAX_ALGO_ORDERS" | 客戶在當前券商上有太多止損/止盈掛單。
"Filter failure: ICEBERG_PARTS" | 冰山訂單可能分成太多部分，`icebergQty`設置的太小。
