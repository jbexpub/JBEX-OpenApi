# 用戶數據流推送

## 基本資訊 

* 壹個用戶數據流的`listenKey`在創建之後的有效期只有60分鐘
* 如果對`listenKey`做`PUT`請求可以延長有效期60分鐘
* 如果對`listenKey`做`DELETE`請求會關閉推送
* 用戶數據流推送在這個端點 **/openapi/ws/<listenKey>** 訪問
* 單壹API連接的有效期只有24小時，請做好在24小時後被斷開連接的準備
* 用戶數據流返回在訂單繁忙期**不保證**順序正常，**請使用E字段進行排序**

## 推送接口

### 創建listenKey

```shell
POST /openapi/v1/userDataStream
```

創建壹個新的用戶數據流。如果沒有發送keepalive，推送將會在60分鐘後斷開。

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

### 延長listenKey有效期

```shell
PUT /openapi/v1/userDataStream
```

發送PUT請求會有效期延長至本次調用後60分鐘，建議每30分鐘發送壹個ping。

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

### 關閉listenKey

```shell
DELETE /openapi/v1/userDataStream
```

關閉用戶數據流。

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

### 賬戶更新

使用`outboundAccountInfo` event進行賬戶更新。

**Payload:**

```javascript
{
  "e": "outboundAccountInfo",   // 事件類型
  "E": 1499405658849,           // 事件時間
  "T": true,                    // 允許交易?
  "W": true,                    // 允許體現?
  "D": true,                    // 允許充值?
  "B": [                        // 余額變化
    {
      "a": "LTC",               // 資產名稱
      "f": "17366.18538083",    // 可用數量
      "l": "0.00000000"         // 凍結數量
    }
  ]
}
```

### 訂單更新

訂單通過`executionReport`事件進行更新。詳細說明資訊請查看 [這裏](Spot%20API.md)。通過將`Z`除以`z`可以找到平均價格。

**Payload:**

```javascript
{
  "e": "executionReport",        // 事件類型
  "E": 1499405658658,            // 事件時間
  "s": "ETHBTC",                 // 交易對
  "c": 1000087761,               // Client order ID
  "S": "BUY",                    // 訂單方向
  "o": "LIMIT",                  // 訂單類型
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // 訂單數量
  "p": "0.10264410",             // 訂單價格
  "X": "NEW",                    // 當前訂單狀態
  "i": 4293153,                  // Order ID
  "l": "0.00000000",             // 最新成交數量
  "z": "0.00000000",             // 累計成交數量
  "L": "0.00000000",             // 最新成交價格
  "n": "0",                      // 手續費
  "N": null,                     // 手續費幣種
  "u": true,                     // 請忽略
  "w": true,                     // 訂單是否工作，適合Stop單
  "m": false,                    // 是否是maker，請忽略
  "O": 1499405658657,            // 訂單創建時間
  "Z": "0.00000000"              // 累計成交數額
}
```

**執行類型:**

* NEW（新訂單）
* PARTIALLY_FILLED（部分成交）
* FILLED（全部成交）
* CANCELED（已撤銷）
* REJECTED（已拒絕）

### 持倉推送

**Payload:**

```javascript
{
  "e": "outboundContractPositionInfo",  // 事件類型
  "A": "",                              // 賬戶ID
  "s": "BTC-SWAP-USDT",                 // symbol
  "S": "LONG",                          // 倉位方向
  "p": "9851.5",                        // 平均價格
  "P": "269",                           // 倉位數量
  "a": "269",                           // 可用
  "f": "7705.9",                        // 強子平倉價
  "m": "59.7884",                       // 保證金
  "r": "-0.0139"                        // 已實現盈虧
}
```