--- 

title: HashKey Custody API Documentation

language_tabs: 
   - shell
   - javascript
   - go
   - java

toc_footers: 
   - <a href='https://github.com/HKDAG/hashkey-custody-api-docs'>Documentation</a>

includes:
  - errors

search: true 

---

# 介绍

Hashkey Custody 提供了一个简单而强大的 RESTful API 和客户端 SDK ，以将数字货币钱包与用户的应用程序集成在一起。欢迎查看我们的 [NodeJs SDK](https://github.com/HKDAG/hashkey-custody-sdk-nodejs.git), [Go SDK](https://github.com/HKDAG/hashkey-custody-sdk-go.git), [Java SDK](https://github.com/HKDAG/jadepool-saas-sdk-java.git)接口。我们提供 2 个级别的 API, 管理API 和 钱包API。

管理 API 可以实现以下功能:

- 创建钱包

钱包 API 可以实现以下功能:

- 钱包余额和交易清单
- 交易创建
- 交易监控和通知

# 环境

HashKey Custody 有两个用于开发和生产的独立环境。 出于安全考虑, 所有API请求均使用TLS加密的HTTPS协议发出。

## 生产环境
生产端点是线上环境，提供给合作伙伴使用。

## 测试环境
测试环境与生产环境完全分开，无论是数据还是账户都没有重叠。

该环境连接到我们支持的各种数字货币的TestNet网络。 这些网络上的代币可以从水龙头上获得，并不代表真实的货币。

# 支持的代币/数字货币
关于生产环境， 请参考文档。
> https://XXXXX

关于测试环境，请参考列表:

| 代币 | 区块链 | 描述 |
| ---- | ---------- | ----------- |
| ETH | Ethereum | Sepolia Testnet |
| USDT | Ethereum | Sepolia Testnet, ERC20 |
| TRX | Tron | Tron Testnet |
| BTC | Bitcoin | public Testnet |

# API 认证



# 一般结构

服务器验证请求的一般请求结构。

**参数**

| 名称 | 位置 | 描述 | 是否必需 | 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| timestamp | query/body | unix timestamp, seconds | Yes | string |
| nonce | query/body | nonce, random string, not repeated in 10 minutes | Yes | string |
| sign | query/body | hex string, sign parameters with HMACSHA256 | Yes | string |

参数 `timestamp`， `nonce` 和 `sign` 位于GET请求的query中, 但位于POST请求的body中。

如果是 POST 请求, body 结构如下:

`
{ 
  "timestamp": 1583376284,
  "nonce": "15833762841261615239762485",
  "mode": "auto",
  "sign":"7042a9fd6deea017be7ad76dfb48e4c36feca279819630c870a628f5352c9044"
}
`

如果是 GET 请求, url 结构如下:

`
/api/v1/app/balance/ETH?timestamp=1583374390&nonce=1583374390314165815853245&sign=7c482cf15d9d25772eee2c43bd146c8136472a055b4587b9fb22d02720037875
`

<font size=4> [签名方式](#signature) 类似于 OAuth.</font>

# 钱包 API

```shell
$ git clone https://github.com/nbltrust/hashkey-custody-sdk-golang.git && cd hashkey-custody-sdk-golang
```

```javascript
const key = 'TyTLvCnHINbWZQag88hhmMz1'
const secret = 'uf0rPlTluGnIllGqx0X1os4hQ6rOdXDxStiN4qGd79lS6yeHZaOK4ldvRv1TBqr6'
const apiAddr = 'http://127.0.0.1:8092'

const api = new API(key, secret, apiAddr)
```

```go
app := sdk.NewAppWithAddr(apiAddr, key, secret)
```

```java
String endpoint = "http://127.0.0.1:8092";
String appKey = "TyTLvCnHINbWZQag88hhmMz1";
String appSecret = "uf0rPlTluGnIllGqx0X1os4hQ6rOdXDxStiN4qGd79lS6yeHZaOK4ldvRv1TBqr6";

APIContext appContext = new APIContext(endpoint, appKey, appSecret);
App appTest = new App(appContext);
```

钱包的 key 和 secret 可在钱包的设置中生成。

出于安全考虑，强烈建议您为 API Key 绑定 IP 地址或 IP 段。

**风险提示：**API Key/Secret 与账号安全紧密相关，无论何时都请勿向其它人透露。API Key/Secret 的泄露可能会造成您的资产损失（即使未开通提现权限），若发现API Key/Secret 泄露请尽快删除该API Key。

## 地址

### 获取最新地址 

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "GetAddress" "ETH" "USDT"
code: 0
message: success
data:
{
  "address": "0x9bf65CDF5729b9588F6bAEBb2Aa2926472D4a035",
  "memo": "",
  "mode": "deposit"
}
```

```javascript
    try {
        const result = await api.getAddress("ETH", "USDT")
        console.log(result)
    } catch(e) {
        // do something
        console.log(e.response)
    }
```

```go
	result, _ := app.GetAddress("ETH", "USDT")
```

```java
  APIResult result = appTest.getAddress("ETH", "USDT");
```

**总结:** 获取最新地址, 若不存在即创建

#### HTTP请求 
`GET /api/v1/address/{networkType}/{coinName}` 

**参数**

| 名称 | 位置 | 描述 | 是否必需 | 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| networkType | path | networkType | Yes | string |
| coinName | path | coinName | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
address | string | the new address
memo | string | address memo
mode | string | address mode

### 创建新地址 

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "CreateAddress" "ETH" "USDT"
code: 0
message: success
data:
{
  "address": "0x9bf65CDF5729b9588F6bAEBb2Aa2926472D4a035",
  "memo": "",
  "mode": "deposit"
}
```

```javascript
    try {
        const result = await api.createAddress("ETH", "USDT")
        console.log(result)
    } catch(e) {
        // do something
        console.log(e.response)
    }
```

```go
	result, _ := app.CreateAddress("ETH", "USDT")
```

```java
  APIResult result = appTest.createAddress("ETH", "USDT");
```

**总结:** 创建新地址

#### HTTP请求
`POST /api/v1/address/{networkType}/{coinName}/new` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| networkType | path | networkType | Yes | string |
| coinName | path | coinName | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
address | string | the new address
memo | string | address memo
mode | string | address mode


## 钱包
### 获取所有可用资产

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "GetAllAssets"
code: 0
message: success
data:
{
  "assets": [{
    "network": "ETH",
    "coin": "USDT"
  }]
}
```

```javascript
    try {
        result = await api.getAllAssets()
        console.log(result)
    } catch(e) {
        // do something
        console.log(e)
    }
```

```go
	result, _ = app.GetAllAssets()
```

```java
  APIResult result = appTest.getAllAssets();
```

**总结:** 获取所有可用资产

#### HTTP请求 
`GET /api/v1/app/allAssets` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
assets | array | the wallet coin list

### 获取钱包余额

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "GetBalances"
code: 0
message: success
data:
{
  "balances": [
    {
      "balance": "10001.225",
      "money": "98175466.911631525",
      "name": "BTC",
      "price": "9816.344189",
      "outLocked": "11.2"
    },
    {
      "balance": "1.0",
      "money": "246.565827",
      "name": "ETH",
      "price": "246.565827",
      "outLocked": "11.2"
    }
  ],
  "total": "98175713.477458525"
}
```

```javascript
    try {
        result = await api.getBalances()
        console.log(result)
    } catch(e) {
        // do something
        console.log(e)
    }
```

```go
	result, _ = app.GetBalances()
```

```java
  APIResult result = appTest.getBalances();
```

**总结:** 获取所有资产余额， 包括以美元计算的数量和价格

#### HTTP请求 
`GET /api/v1/app/balances` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
total | string | total money(USD)
balances | array | the wallet balance list

余额:

值 | 类型 | 描述
--------- | ------- | ---------
balance | string | 币种余额
money | string | 币种余额对应的美元价值
name | string | 币种名称
price | string | 币种价格(USD)
outLocked | string | 币种转出锁定余额

### 获取钱包订单

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "GetOrders" 1 10
code: 0
message: success
data:
{
  "totalAmount": 1,
  "orders": [{
    "bizType": "WITHDRAW",
    "block": 12970670,
    "coinName": "USDT",
    "confirmations": 21,
    "fee": "0.005000000000000000",
    "from": "0xdedc1eca923cc1227c20571030146d8a01b70774",
    "id": "rNXBQGJlw09apVyg4nDo",
    "memo": "",
    "n": 0,
    "state": "DONE",
    "to": "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072",
    "txid": "0xcfc4cb925c77c2ea10be3c233029fc1f05d0ddffe7396920ddd493544e52933d",
    "type": "ETH",
    "value": "0.045000000000000000"
  }]
}
```

```javascript
    try {
        result = await api.getOrders(1, 10)
        console.log(result)
    } catch(e) {
        // do something
        console.log(e)
    }
```

```go
	result, _ = app.GetOrders(1, 10)
```

```java
  APIResult result = appTest.getOrders(1, 10);
```

**总结:** 获取钱包订单

#### HTTP请求 
`GET /api/v1/app/orders` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| page | query | page, e.g. 1 | Yes | string |
| amount | query | item count on this page, e.g. 10 | Yes | string |
| coins | query | coin list, can be empty string | No | string |
| state | query | order state, can be empty string | No | string |
| bizType | query | order type, TRANSFER_IN/TRANSFER_OUT/WITHDRAW/DEPOSIT | No | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
orders | array | order list
totalAmount | number | the total count of orders

order:


值 | 类型 | 描述
--------- | ------- | ---------
bizType | string | 订单类型
block | number | 区块高度
coinName | string | 币种名称
confirmations | number | 交易确认数
fee | string | 交易费用
from | string | 交易输入
id | string | 订单ID
memo | string | memo
n | number | 订单索引
state | string | 订单状态
to | string | 交易输出
txid | string | 交易哈希
type | string | 网络类型
value | string | 交易值
note | string | 订单备注
message | string | 转账消息
createdAt | number | 创建时间(unix时间戳, 秒)
relatedOrderId | string | 关联订单ID

**`state` 状态枚举**

值 | 描述 | 是否终态
--------- | ------- | ---------
WAITING | 订单等待审核/审批 | 否
PENDING | 订单已提交，正在链上确认 | 否
DEPOSIT_FAIL | 仅充值：充值入账失败 | 否
ADDRESS_VERIFYING | 仅充值：正在校验充值地址 | 否
KYT_REVIEWING | 仅充值：充值正在进行 KYT 审查 | 否
MANUAL_AML | 仅充值：充值正在进行人工 KYT 审查 | 否
INCREASING_AMOUNT | 仅充值：正在为充值入账 | 否
REFUNDING | 仅充值：充值正在退款中 | 否
REFUND | 仅充值：充值已退款 | 是
DONE | 订单成功完成 | 是
TERMINATED | 订单被终止（例如审核被拒） | 是
FAILED | 订单失败 | 是

### 获取钱包单笔订单

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "GetOrder" "rNXBQGJlw09apVyg4nDo"
code: 0
message: success
data:
{
  "bizType": "WITHDRAW",
  "block": 12970670,
  "coinName": "USDT",
  "confirmations": 21,
  "fee": "0.005000000000000000",
  "from": "0xdedc1eca923cc1227c20571030146d8a01b70774",
  "id": "rNXBQGJlw09apVyg4nDo",
  "memo": "",
  "n": 0,
  "state": "DONE",
  "to": "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072",
  "txid": "0xcfc4cb925c77c2ea10be3c233029fc1f05d0ddffe7396920ddd493544e52933d",
  "type": "ETH",
  "value": "0.045000000000000000",
  "note": "note",
  "message": ""
}
```

```javascript
    try {
        result = await api.getOrder("rNXBQGJlw09apVyg4nDo")
        console.log(result)
    } catch(e) {
        // do something
        console.log(e)
    }
```

```go
	result, _ = app.GetOrder("rNXBQGJlw09apVyg4nDo")
```

```java
  APIResult result = appTest.getOrder("rNXBQGJlw09apVyg4nDo");
```

**总结:** 获取钱包单笔订单

#### HTTP请求 
`GET /api/v1/app/order/{id}` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| id | path | order id | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
bizType | string | 订单类型
block | number | 区块高度
coinName | string | 币种名称
confirmations | number | 交易确认数
fee | string | 交易费用
from | string | 交易输入
id | string | 订单ID
memo | string | memo
n | number | 订单索引
state | string | 订单状态
to | string | 交易输出
txid | string | 交易哈希
type | string | 网络类型
value | string | 交易值
note | string | 订单备注
message | string | 转账消息
createdAt | number | 创建时间(unix时间戳, 秒)
relatedOrderId | string | 关联订单ID

**`state` 状态枚举**

值 | 描述 | 是否终态
--------- | ------- | ---------
WAITING | 订单等待审核/审批 | 否
PENDING | 订单已提交，正在链上确认 | 否
DEPOSIT_FAIL | 仅充值：充值入账失败 | 否
ADDRESS_VERIFYING | 仅充值：正在校验充值地址 | 否
KYT_REVIEWING | 仅充值：充值正在进行 KYT 审查 | 否
MANUAL_AML | 仅充值：充值正在进行人工 KYT 审查 | 否
INCREASING_AMOUNT | 仅充值：正在为充值入账 | 否
REFUNDING | 仅充值：充值正在退款中 | 否
REFUND | 仅充值：充值已退款 | 是
DONE | 订单成功完成 | 是
TERMINATED | 订单被终止（例如审核被拒） | 是
FAILED | 订单失败 | 是

### 更新钱包订单

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "UpdateOrder" "rNXBQGJlw09apVyg4nDo" "123"
code: 0
message: success
data:
{}
```

```go
	result, _ = app.UpdateOrder("rNXBQGJlw09apVyg4nDo", "123")
```

**总结:** 更新钱包订单

#### HTTP请求 
`PUT /api/v1/app/order/{id}` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| id | path | order id | Yes | string |
| note | body | note | No | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------

### 提现

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "Withdraw" "$(date +%s)" "ETH" "USDT" "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072" "0.05"
code: 0
message: success
data:
{
  "bizType": "WITHDRAW",
  "block": -1,
  "coinName": "USDT",
  "confirmations": 0,
  "fee": "0.0050000000",
  "from": "",
  "id": "Jd7qbDRa1qM1lxK5Qe2M",
  "memo": "",
  "n": 0,
  "state": "INIT",
  "to": "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072",
  "txid": "",
  "type": "ETH",
  "value": "0.0450000000"
}
```

```go
	result, _ = app.Withdraw("1569225735", "ETH", "USDT", "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072", "0.05")
```

**总结:** 提现

#### HTTP请求 
`POST /api/v1/app/{network}/{coinName}/withdraw` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| coinName | path | coin type | Yes | string |
| id | body | withdraw id | Yes | string |
| to | body | receive address | Yes | string |
| value | body | withdraw amount | Yes | string |
| memo | body | address memo | No | string |
| note | body | note | No | string |
| priority | body | priority | No | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
bizType | string | order type
block | number | the block transaction mined in
coinName | string | token name
confirmations | number | number of transaction confirmations
fee | string | fee burnt for the transaction
from | string | transaction input
id | string | order id
memo | string | address memo
n | number | order index
state | string | order state
to | string | transaction output
txid | string | transaction hash
type | string | network type
value | string | transaction value
note | string | order note

### 划转

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" Transfer "$(date +%s)" "e5dJyVp8R3B1m4o" "ETH" "0.01"
code: 0
message: success
data:
{
  "bizType": "TRANSFER_OUT",
  "coinName": "ETH",
  "confirmations": 0,
  "fee": "0",
  "from": "L6RayqPn4jXExW0",
  "id": "orders4eq8kz1235jz2yj0n9rvpwl7",
  "memo": "",
  "message": "",
  "n": 0,
  "note": "",
  "state": "DONE",
  "to": "e5dJyVp8R3B1m4o",
  "txid": "",
  "type": "ETH",
  "value": "0.01"
}
```

```go
	result, _ = app.Transfer("1569225735", "e5dJyVp8R3B1m4o", "ETH", "0.01")
```

**总结:** 划转

#### HTTP请求 
`POST /api/v1/app/{coinName}/transfer` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |
| coinName | path | coin type | Yes | string |
| id | body | transfer id | Yes | string |
| to | body | receive wallet id | Yes | string |
| value | body | transfer amount | Yes | string |
| note | body | transfer note, can be empty string | Yes | string |
| message | body | transfer message, can be empty string | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
bizType | string | order type
block | number | the block transaction mined in
coinName | string | unique token name
confirmations | number | number of transaction confirmations
fee | string | fee burnt for the transaction
from | string | transaction input
id | string | order id
memo | string | order note
n | number | order index
state | string | order state
to | string | transaction output
txid | string | transaction hash
value | string | transaction value
note | string | transfer note
message | string | transfer message

## 系统

### 当前时间戳

```shell
$ go run cmd/ctl/main.go "appkey" "appsecret" "SystemGetTime"
code: 0
message: success
data:
{
  "timestamp": 1590391287
}
```

**总结:** 获取系统当前时间戳

#### HTTP请求 
`GET /api/v1/system/time`

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-App-Key | header | app key | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
timestamp | number | current timestamp

# 企业 API

```shell
$ git clone https://github.com/nbltrust/hashkey-custody-sdk-golang.git && cd hashkey-custody-sdk-golang
```

```javascript
const companyKey = 'TyTLvCnHINbWZQag88hhmMz1'
const companySecret = 'uf0rPlTluGnIllGqx0X1os4hQ6rOdXDxStiN4qGd79lS6yeHZaOK4ldvRv1TBqr6'
const apiAddr = 'http://127.0.0.1:8092'

const api = new API(companyKey, companySecret, apiAddr)
```

```go
company := sdk.NewCompanyWithAddr(apiAddr, key, secret)
```

```java
String endpoint = "http://127.0.0.1:8092";
String appKey = "TyTLvCnHINbWZQag88hhmMz1";
String appSecret = "uf0rPlTluGnIllGqx0X1os4hQ6rOdXDxStiN4qGd79lS6yeHZaOK4ldvRv1TBqr6";
APIContext companyContext = new APIContext(endpoint, appKey, appSecret);
Company companyTest = new Company(companyContext);
```

企业 key 和 secret 可在企业设置中生成。

出于安全考虑，强烈建议您为企业 API Key 绑定  IP 地址或 IP 段。

**风险提示：**API Key/Secret 与账号安全紧密相关，无论何时都请勿向其它人透露。若发现API Key/Secret 泄露请尽快删除该API Key。

## 钱包
### 创建

```shell
$ go run cmd/ctl/main.go "companykey" "companysecret" "CreateWallet" "name" "" "https://noti.domain/callback"
code: 0
message: success
data:
{
  "id": "ylr07eg5p862mkpw",
  "appKey": "52u036rjmnd5qtwc74vohpj3",
  "appSecret": "4d6718b405cb55e5544ba0b343472acba8a5a1eb0c5777e4040b38f9edd96a60",
  "encryptedAppSecret": "7p128MoVWgSFI7AYAUXO9Px72qCNYSvtM5PCAGD3f6de+HjYJbJBEJcTswMH1qdLcKIriuy1RoP6P0YmNeEDlAfSSyJiHrX98Oid5/fuCEs="
}
```

```go
	result, _ = company.CreateWallet("name", "", "https://noti.domain/callback")
```

```java
  APIResult result = companyTest.createWallet("name", "", "https://noti.domain/callback", "description");
```

**总结:** 创建钱包

#### HTTP请求 
`POST /api/v1/app` 

**参数**

| 名称 | 位置 | 描述| 是否必需| 类型 |
| ---- | ---------- | ----------- | -------- | ---- |
| X-Company-Key | header | company key | Yes | string |
| name | body | the wallet name | Yes | string |
| description | body | the wallet description | Yes | string |
| webHook | body | the url will be called when send [a notification](#callback) | Yes | string |
| aesIV | body | base64 encoded [AES encryption](#aes-encryption) iv, must be 16 bytes, used by encrypting app password and secret | Yes | string |

**响应结果**

值 | 类型 | 描述
--------- | ------- | ---------
id | string | wallet id
appKey | string | the appkey placed in the header of [Wallet API](#wallet-api)
encryptedAppSecret | string | the base64 encoded [AES encrypted](#aes-encryption) appsecret, the appsecret used for the [signature](#signature) of [Wallet API](#wallet-api)

# 回调

## 订单
> 订单通知发送的 JSON 结构如下:

```json
{
  "id": "2XB0eKAvj7KvjZDk59zl",
  "withdrawID": "7Cab38EA42538f4D8C2",
  "bizType": "DEPOSIT",
  "coinName": "ETH",
  "state": "DONE",
  "value": "1.000000000000000000",
  "fee": "0.000000000000000000",
  "from": "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072",
  "to": "0x29152c850456899A78178622B6543BBFfC224495",
  "txid": "0x8487e23bbf71f1763e015598283ae891cc5ea8d444f87a0a60a0b5eb7e1a4d59",
  "confirmations": 27,
  "sign": "796dde931a15c98edc3dfdb65a2c2addfde422f217a1f6934be9226542839aa0"
}
```

**回调结果**

值 | 类型 | 描述
--------- | ------- | ---------
id | string | 订单ID
withdrawID | string | 提币ID
coinName | string | 币种名称
txid | string | 交易哈希
state | string | 订单状态
bizType | string | 订单类型
from | string | 交易输入，提币订单该值为空
to | string | 交易输出
value | string | 交易值
confirmations | number | 交易确认数
fee | string | 提现手续费
sign | string | 签名

**`state` 状态枚举**

值 | 描述 | 是否终态
--------- | ------- | ---------
WAITING | 订单等待审核/审批 | 否
PENDING | 订单已提交，正在链上确认 | 否
DEPOSIT_FAIL | 仅充值：充值入账失败 | 否
ADDRESS_VERIFYING | 仅充值：正在校验充值地址 | 否
KYT_REVIEWING | 仅充值：充值正在进行 KYT 审查 | 否
MANUAL_AML | 仅充值：充值正在进行人工 KYT 审查 | 否
INCREASING_AMOUNT | 仅充值：正在为充值入账 | 否
REFUNDING | 仅充值：充值正在退款中 | 否
REFUND | 仅充值：充值已退款 | 是
DONE | 订单成功完成 | 是
TERMINATED | 订单被终止（例如审核被拒） | 是
FAILED | 订单失败 | 是

### 签名
上面的 sign 参数可以通过下面的方法验证，以保证这个回调结果的发送方身份:

1. 形成一个字符串消息（排序），其中包含数据prarams（排除 sign）。

    如果body params是: 
</br>
`
{
  "id": "2XB0eKAvj7KvjZDk59zl",
  "withdrawID": "7Cab38EA42538f4D8C2",
  "bizType": "DEPOSIT",
  "coinName": "ETH",
  "state": "DONE",
  "value": "1.000000000000000000",
  "fee": "0.000000000000000000",
  "from": "0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072",
  "to": "0x29152c850456899A78178622B6543BBFfC224495",
  "txid": "0x8487e23bbf71f1763e015598283ae891cc5ea8d444f87a0a60a0b5eb7e1a4d59",
  "confirmations": 27
}
`

    消息字符串如下:
</br>
`
affirmativeConfirmation=20&bizType=DEPOSIT&block=13721091&coinName=ETH&confirmations=27&fee=0.000000000000000000&from=0xF0706B7Cab38EA42538f4D8C279B6F57ad1d4072&id=2XB0eKAvj7KvjZDk59zl&memo=&n=0&state=DONE&to=0x29152c850456899A78178622B6543BBFfC224495&txid=0x8487e23bbf71f1763e015598283ae891cc5ea8d444f87a0a60a0b5eb7e1a4d59&type=ETH&value=1.000000000000000000&withdrawID=7Cab38EA42538f4D8C2
`

2. 使用HMAC-SHA256对消息进行签名。如果 AppSecret 是 `exzYZT8IubM9Jxq1PWU5QjZ0JFP81bvCmlf1fFjW0b87Zr6eAMdofeYlhAMZxPzo`, 那么sign param 如下:
</br>
`
fb0f53f33bba4cfa4bcb2c81e976bbe817633ba87a9904b6c3de293da3805cb3
`

# 签名
1. G获取当前的时间戳（秒）和nonce（随机字符串）。请确保时间戳错误不超过5分钟，nonce不在10分钟内重复。

2. 形成一个字符串消息（排序），其中包含数据prarams，上述时间戳和随机字符串。

    如果body params是: 
</br>
`
{
  "mode": "auto"
}
`

    消息字符串如下:
</br>
`
mode=auto&nonce=15833762841261615239762485&timestamp=1583376284
`

3. 使用HMAC-SHA256对消息进行签名。如果 AppSecret 是 `yeTJ3EnOkyQQEjhTMVqn165Dqjp43bhTwXLIv25Ycdu8qwDOyqpa0WV54C6sO4HW`, 那么sign param 如下:
</br>
`
7042a9fd6deea017be7ad76dfb48e4c36feca279819630c870a628f5352c9044
`

4. 发送请求的格式与 [一般结构](#general-structure)中所述的相同。

# AES 加密
加密有两部分: key 和 iv。 key 是 sha256(CompanySecret)， iv 是一个长度为16的随机字节。以下为用python编写的加密appSecret的例子。

```python
import base64
import hashlib
from Crypto.Cipher import AES

def _pad(s):
    return s + (AES.block_size - len(s) % AES.block_size) * chr(AES.block_size - len(s) % AES.block_size)

def _unpad(s):
    return s[:-ord(s[len(s)-1:])]

companySecret = b'kfTKfuSV5oiuFZENI6tv1OwfiJ1tEv70HnmJsVxNGCu2YEKqEq2QHpBGgqwRa29R'
appSecret = 'nn2oh58iuc1sg2adya1golz35cowjbz7r0hfjt2ey6b4fc4xyq6kdopp9tlp0ko3'

aeskey = bytes.fromhex(hashlib.sha256(companySecret).hexdigest())
iv = base64.b64decode('MTIzNDU2Nzg5MDEyMzQ1Ng==')

decipher = AES.new(aeskey, AES.MODE_CBC, IV=iv)
encryptedAppSecret = decipher.encrypt(_pad(appSecret))
base64EncryptedAppSecret = base64.b64encode(encryptedAppSecret)
print(base64EncryptedAppSecret)

decipher = AES.new(aeskey, AES.MODE_CBC, IV=iv)
plaintext = _unpad(decipher.decrypt(base64.b64decode(base64EncryptedAppSecret)))
print(plaintext)
```

# ECC 签名
>  Javascript code of building the message to be signed:

```js
function _buildMsg (obj, opts = {}) {
  let sortRule = opts.sort || 'key-alphabet'
  let arr = []
  if (_.isArray(obj)) {
    arr = obj.map((o, i) => ({
      k: (sortRule === 'kvpair' || sortRule === 'value') ? '' : i,
      v: _buildMsg(o, opts)
    }))
  } else if (_.isObject(obj)) {
    for (let k in obj) {
      if (obj[k] !== undefined) {
        arr.push({ k, v: _buildMsg(obj[k], opts) })
      }
    }
  } else if (obj === undefined || obj === null) {
    return ''
  } else {
    return obj.toString()
  }
  // Sort Array
  arr.sort((a, b) => {
    let aVal
    let bVal
    switch (sortRule) {
      case 'key':
        aVal = a.k
        bVal = b.k
        break
      case 'key-alphabet':
        aVal = a.k.toString()
        bVal = b.k.toString()
        break
      case 'value':
        aVal = a.v
        bVal = b.v
        break
      case 'kvpair':
      default:
        aVal = a.k.toString() + a.v
        bVal = b.k.toString() + b.v
        break
    }
    if (aVal < bVal) {
      return -1
    } else if (aVal === bVal) {
      return 0
    } else {
      return 1
    }
  })
  // Build message
  return arr.reduce((lastMsg, curr) => {
    return lastMsg + curr.k + curr.v
  }, '')
}
```

*构建 GET 请求的签名步骤::*

1. 获取当前的时间戳
2. 将时间戳和其他所有请求参数按字母排序生成一个字符串，字符串像下面这样:
</br>
```
timestamp1557913602438
```
3. 使用 sha3 keccak256 编码上面的字符串
4. 使用私钥通过 ecdsa 签名上面编码好的字符串
5. 将签名结果的 r 和 s 进行 base64 编码后放入请求参数 sigR 和 sigS 中，时间戳放入 timestamp 中，假设私钥(hex编码)为 1cc64d8767ff6fe618a7794d1e16cced03ab35715d58d48029de87b475abef85 ，那么最终的 sigR 和 sigS 为
</br>
```
sigR = '2vS/6a+4+X0wyDyqAw5gTcLS02cCvH+CsSDIOsStPvc='
sigS = 'S7TIyw5oYkoBUC0mBUIHxabMU4slREcMAMYo74ZO2wM='
```

*构建 POST/PUT 请求的签名步骤:*

1. 获取当前的时间戳
2. 将body中的所有请求参数和时间戳按字母排序生成一个字符串（右侧是js的示例代码），以交换币种接口为例（{sequence: 'abcd', from: 'walletwovldjmmmqj2m0n9', to: 'walletp2q8yjerrr6owxmr', fromAssetID: 1, fromAmount: '123', toAssetID: 2, toAmount: '321', note: 'test sig', timestamp: 1557913602438}），字符串像下面这样:
</br>
```
fromwalletwovldjmmmqj2m0n9fromAmount123fromAssetID1notetest sigsequenceabcdtimestamp1557913602438towalletp2q8yjerrr6owxmrtoAmount321toAssetID2
```
3. 使用 sha3 keccak256 编码上面的字符串
4. 使用私钥通过 ecdsa 签名上面编码好的字符串
5. 将签名结果的 r 和 s 进行 base64 编码后放入请求参数 sig 中，时间戳放入 timestamp 中，假设私钥(hex编码)为 1cc64d8767ff6fe618a7794d1e16cced03ab35715d58d48029de87b475abef85 ，那么最终的 sig 为
</br>
```
sig: {
  r: 'Il4vOx0mIfQLRwFQt9E4480IsuwyCvPOHvTxWnv+CM0=',
  s: 'Psdc5NI36LE+R/X6qVvumtjTvo0ufqBBqEBB62FRTh0='
}
```
