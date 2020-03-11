# 用户数据流推送

## 基本信息 

* 一个用户数据流的`listenKey`在创建之后的有效期只有60分钟
* 如果对`listenKey`做`PUT`请求可以延长有效期60分钟
* 如果对`listenKey`做`DELETE`请求会关闭推送
* 用户数据流推送在这个端点 **/openapi/ws/<listenKey>** 访问
* 单一API连接的有效期只有24小时，请做好在24小时后被断开连接的准备
* 用户信息流返回在订单繁忙期**不保证**顺序正常，**请使用E字段进行排序**

## 推送接口

### 创建listenKey

```shell
POST /openapi/v1/userDataStream
```

创建一个新的用户信息流。如果没有发送keepalive，推送将会在60分钟后断开。

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

### 延长listenKey有效期

```shell
PUT /openapi/v1/userDataStream
```

发送PUT请求会有效期延长至本次调用后60分钟，建议每30分钟发送一个ping。

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

### 关闭listenKey

```shell
DELETE /openapi/v1/userDataStream
```

关闭用户数据流。

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

## Web Socket Payloads

### 账户更新

使用`outboundAccountInfo` event进行账户更新。

**Payload:**

```javascript
{
  "e": "outboundAccountInfo",   // 事件类型
  "E": 1499405658849,           // 事件时间
  "T": true,                    // 允许交易?
  "W": true,                    // 允许体现?
  "D": true,                    // 允许充值?
  "B": [                        // 余额变化
    {
      "a": "LTC",               // 资产名称
      "f": "17366.18538083",    // 可用数量
      "l": "0.00000000"         // 冻结数量
    }
  ]
}
```

### 订单更新

订单通过`executionReport`事件进行更新。详细说明信息请查看 [这里](Spot%20API.md)。通过将`Z`除以`z`可以找到平均价格。

**Payload:**

```javascript
{
  "e": "executionReport",        // 事件类型
  "E": 1499405658658,            // 事件时间
  "s": "ETHBTC",                 // 交易对
  "c": 1000087761,               // Client order ID
  "S": "BUY",                    // 订单方向
  "o": "LIMIT",                  // 订单类型
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // 订单数量
  "p": "0.10264410",             // 订单价格
  "X": "NEW",                    // 当前订单状态
  "i": 4293153,                  // Order ID
  "l": "0.00000000",             // 最新成交数量
  "z": "0.00000000",             // 累计成交数量
  "L": "0.00000000",             // 最新成交价格
  "n": "0",                      // 手续费
  "N": null,                     // 手续费币种
  "u": true,                     // 请忽略
  "w": true,                     // 订单是否工作，适合Stop单
  "m": false,                    // 是否是maker，请忽略
  "O": 1499405658657,            // 订单创建时间
  "Z": "0.00000000"              // 累计成交数额
}
```

**执行类型:**

* NEW（新订单）
* PARTIALLY_FILLED（部分成交）
* FILLED（全部成交）
* CANCELED（已撤销）
* REJECTED（已拒绝）

### 持仓推送

**Payload:**

```javascript
{
  "e": "outboundContractPositionInfo",  // 事件类型
  "A": "",                              // 账户ID
  "s": "BTC-SWAP-USDT",                 // symbol
  "S": "LONG",                          // 仓位方向
  "p": "9851.5",                        // 平均价格
  "P": "269",                           // 仓位数量
  "a": "269",                           // 可用
  "f": "7705.9",                        // 强子平仓价
  "m": "59.7884",                       // 保证金
  "r": "-0.0139"                        // 已实现盈亏
}
```