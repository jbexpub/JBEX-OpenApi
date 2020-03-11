# 错误码

返回报错一般由两个部分组成：错误码和错误信息。错误码是通用的，但是错误信息会有所不同。如下是一个报错JSON Payload示例：

```javascript
{
  "code":-1121,
  "msg":"Invalid symbol."
}
```

## 10xx - 通用服务器和网络错误

### -1000 UNKNOWN

* 处理请求时发生未知错误

### -1001 DISCONNECTED

* 内部错误; 无法处理您的请求。 请再试一次.

### -1002 UNAUTHORIZED

* 您无权执行此请求。请求需要发送API Key，我们建议在所有的请求种豆附加API Key 

### -1003 TOO_MANY_REQUESTS

* 排队的请求过多。  
* 请求权重过多； 请使用websocket进行实时更新。  
* 请求权重过多； 当前限制为每分钟％s请求权重。 请使用websocket进行实时更新，以避免轮询API
* 请求权重过多； IP被禁止，直到％s。 请使用websocket进行实时更新，以免被禁。

### -1006 UNEXPECTED_RESP

* 从消息总线收到意外的响应。 执行状态未知。请向客服求证关于此订单的详细状态和其他信息。

### -1007 TIMEOUT

* 等待后端服务器响应超时。 发送状态未知； 执行状态未知。

### -1014 UNKNOWN_ORDER_COMPOSITION

* 不支持的订单组合。

### -1015 TOO_MANY_ORDERS

* 新订单太多。请减少你的请求频率
* 新订单太多； 当前限制为每％s ％s个订单。

### -1016 SERVICE_SHUTTING_DOWN

* 该服务不可用

### -1020 UNSUPPORTED_OPERATION

* 不支持此操作

### -1021 INVALID_TIMESTAMP

* 此请求的时间戳不在recvWindow之外。
* 此请求的时间戳比服务器时间提前1000毫秒。
* 请查证你的本地时间和服务器时间

### -1022 INVALID_SIGNATURE

* 此请求的签名无效。

## 11xx - 请求问题

### -1100 ILLEGAL_CHARS

* 在参数中发现非法字符。 
* 在参数'％s'中发现非法字符； 合法范围是“％s”。

### -1101 TOO_MANY_PARAMETERS

* 为此端点发送的参数太多。  
* 参数太多； 预期为“％s”并收到了“％s”。 
* 检测到的参数值重复。

### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED

* 未发送强制性参数，该参数为空/空或格式错误。  
* 强制参数'％s'未发送，为空/空或格式错误。  
* 必须发送参数'％s'或'％s'，但两者均为空！

### -1103 UNKNOWN_PARAM

* 发送了未知参数。
* 每条请求需要至少一个参数{Timestamp}.

### -1104 UNREAD_PARAMETERS

* 并非所有发送的参数都被读取。
* 并非所有发送的参数都被读取； 读取了'％s'参数，但被发送了'％s'。

### -1105 PARAM_EMPTY

* 参数为空。
* 参数'％s'为空。

### -1106 PARAM_NOT_REQUIRED

* 不需要时已发送参数。
* 不需要时发送参数'％s'。

### -1111 BAD_PRECISION

* 精度超过为此资产定义的最大值。

### -1112 NO_DEPTH

* 交易对没有挂单

### -1114 TIF_NOT_REQUIRED

* 不需要时发送了TimeInForce参数。

### -1115 INVALID_TIF

* 无效 timeInForce.

### -1116 INVALID_ORDER_TYPE

* 无效订单类型。

### -1117 INVALID_SIDE

* 无效买卖方向。

### -1118 EMPTY_NEW_CL_ORD_ID

* 新的客户订单ID为空。

### -1119 EMPTY_ORG_CL_ORD_ID

* 原始客户订单ID为空。

### -1120 BAD_INTERVAL

* 无效时间间隔。

### -1121 BAD_SYMBOL

* 无效符号。

### -1125 INVALID_LISTEN_KEY

* 该listenKey不存在。

### -1127 MORE_THAN_XX_HOURS

* 查询间隔太大。
* 从开始时间到结束时间之间超过％s小时。

### -1128 OPTIONAL_PARAMS_BAD_COMBO

* 可选参数组合无效。

### -1130 INVALID_PARAMETER

* 发送的参数为无效数据。
* 发送参数'％s'的数据无效。

### -1132 ORDER_PRICE_TOO_HIGH

* 订单价格过高

### -1133 ORDER_PRICE_TOO_SMALL

* 订单价格过低，请查询Broker Info信息

### -1134 ORDER_PRICE_PRECISION_TOO_LONG

* 订单价格精度过长，请查询Broker Info信息

### -1135 ORDER_QUANTITY_TOO_BIG

* 订单quantity过大

### -1136 ORDER_QUANTITY_TOO_SMALL

* 订单quantity小于最小值

### -1137 ORDER_QUANTITY_PRECISION_TOO_LONG

* 订单quantity精度过长

### -1138 ORDER_PRICE_WAVE_EXCEED

* 订单价格超出允许范围

### -1139 ORDER_HAS_FILLED

* 订单已经被执行

### -1140 ORDER_AMOUNT_TOO_SMALL

* 交易金额小于最小值

### -1141 ORDER_DUPLICATED

* clientOrderId重复

### -1142 ORDER_CANCELLED

* 订单已经被撤销

### -1143 ORDER_NOT_FOUND_ON_ORDER_BOOK

* 订单当前不在订单簿上

### -1144 ORDER_LOCKED

* 订单已被锁定

### -1145 ORDER_NOT_SUPPORT_CANCELLATION

* 该订单类型不支持撤销

### -1146 ORDER_CREATION_TIMEOUT

* 订单生成超时

### -1147 ORDER_CANCELLATION_TIMEOUT

* 订单撤销超时

### -2010 NEW_ORDER_REJECTED

* NEW_ORDER_REJECTED

### -2011 CANCEL_REJECTED

* CANCEL_REJECTED

### -2013 NO_SUCH_ORDER

* Order不存在

### -2014 BAD_API_KEY_FMT

* API-key格式无效

### -2015 REJECTED_MBX_KEY

* 无效的API密钥，IP或操作权限。.

### -2016 NO_TRADING_WINDOW

* 找不到该交易对的交易窗口。 尝试改为24小时自动报价。

## -1010 ERROR_MSG_RECEIVED, -2010 NEW_ORDER_REJECTED, -2011 CANCEL_REJECTED的错误信息

匹配引擎返回错误后，将发送此代码。 以下消息将指示特定的错误：

Error message | Description
------------ | ------------
"Unknown order sent." | 找不到订单（通过`orderId`，`clientOrderId`，`origClientOrderId`）
"Duplicate order sent." | `clOrdId`已经被使用
"Market is closed." | 该交易对不在交易范围
"Account has insufficient balance for requested action." | 没有足够的资金来完成行动
"Market orders are not supported for this symbol." | 交易对上未启用`MARKET`
"Iceberg orders are not supported for this symbol." | 该交易对未启用`icebergQty`
"Stop loss orders are not supported for this symbol." | 该交易对未启用`STOP_LOSS`
"Stop loss limit orders are not supported for this symbol." | 该交易对未启用`STOP_LOSS_LIMIT`
"Take profit orders are not supported for this symbol." | 该交易对未启用`TAKE_PROFIT`
"Take profit limit orders are not supported for this symbol." | 该交易对未启用`TAKE_PROFIT_LIMIT`
"Price* QTY is zero or less." | `price`* `quantity`太小
"IcebergQty exceeds QTY." | `icebergQty` 必须小于订单数量
"This action disabled is on this account." | 联系客户支持；该帐户已禁用了某些操作。
"Unsupported order combination" | 不允许组合`orderType`, `timeInForce`, `stopPrice`, 和/或 `icebergQty`。
"Order would trigger immediately." | 与最后交易价格相比，订单的止损价无效。
"Cancel order is invalid. Check origClOrdId and orderId." | 未发送`origClOrdId` 或 `orderId`。
"Order would immediately match and take." | `LIMIT_MAKER`订单类型将立即匹配并进行交易，而不是纯粹的生成订单。 

## -9xxx 过滤器故障

Error message | Description
------------ | ------------
"Filter failure: PRICE_FILTER" | `价格`过高，过低和/或不遵循交易对的最小价格规则。
"Filter failure: LOT_SIZE" | `数量`太高，太低和/或不遵循该交易对的步长规则。 
"Filter failure: MIN_NOTIONAL" | `价格` * `数量`太低，无法成为该交易对的有效订单 
"Filter failure: MAX_NUM_ORDERS" | 客户在交易对上有太多挂单。
"Filter failure: MAX_ALGO_ORDERS" | 客户在交易对上有太多止损/止盈挂单。
"Filter failure: BROKER_MAX_NUM_ORDERS" | 客户在当前券商上有太多挂单。
"Filter failure: BROKER_MAX_ALGO_ORDERS" | 客户在当前券商上有太多止损/止盈挂单。
"Filter failure: ICEBERG_PARTS" | 冰山订单可能分成太多部分，`icebergQty`设置的太小。