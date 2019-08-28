# MorningPay 开发者接口 
```当前版本：1.0```

| 版本历史      | 备注               |时间                                          |
| ---------- | -------------------           |           ---|
| 1.0 | 完成EOS钱包功能               |  2019/01/01  |

测试地址TEST_URL:``

测试APP_KEY: ``


测试APP_SECRET:
``


## 1.签名


### 如何签名 (验证)

在给一个API请求签名之前, 你必须准备好你的`app_key/app_secret`。

在注册并认证通过后之后，只需访问API密钥页面就可以得到您的密钥。

所有的API都需要使用`http_query`拼接成的字符串与`app_secret`这2个参数生成的`x-sign`与我们提供的`x-appkey`用于身份验证。实际代码操作中`x-sign `与`x-appkey`需要被添加到`http.Header`（具体可见下方完整代码示例）中。

| 参数       | 解释                                                         |
| ---------- | ---------------------------------------------  |
| x-sign |你的`app_key`                                   |
| x-sign| 使用`http_query拼接成的字符串`与`app_secret`生成的签名 |

代码片段可能是这样：

```

appKey = "你的appkey"
appSecret = "你的appSecret"
httpQuery="Method|Path|经过排序的params"
xHermesSignature = HmacSHA256(httpQuery,appSecret)

req,_ = http.NewRequest("POST", grpcHttpUrl, nil)
req.Header.Set("x-appkey", appKey)
req.Header.Set("x-sign", xHermesSignature)


```


#### 签名步骤

1.通过组合HTTP方法, 请求地址和请求参数得到 `payload`。

需要注意的是：参数会进行排序，排序规则是 ：按照字母序排列，排列方式是 `a->z`，从第一个字母开始。如果首字母相同，则会对第二个字母进行排序，以此类推。

~~~
# canonical_verb 是HTTP方法，例如 POST
# canonical_uri 是请求地址， 例如 /wallet/addresses
# canonical_query 是请求参数通过&连接而成的字符串，参数包括http请求方式请求路径与请求参数, 参数必须按照字母序排列，排列方式是 `a->z` ，例如 coin_type=BTC
# 最后再把这三个字符串通过'|'字符连接起来，看起来就像这样:
# POST|/wallet/addresses|coin_type=BTC
#
def payload
  "#{canonical_verb}|#{canonical_uri}|#{canonical_query}"
end

~~~

2. 对上述字符串使用`HMAC-SHA256`加密算法进行SHA256计算:

~~~
 x-appkey = HMAC-SHA256(payload, app_secret)
 处理得到：
 x-sign = 7e83208c3d14817849716032612de11b1a400892130a9dabbd63d356e2aabf72
~~~

推荐使用一个在线工具：

`http://tool.oschina.net/encrypt?type=2`


#### 一个完整的范例

npm install crypto-js
~~~
import HmacSHA256 from 'crypto-js/hmac-sha256'

getAddresses(){
    let appkey = "39dd949c-f748-4f9a-9620-2d7cb8f850a5";
    var signStr = "GET|/wallet/address|BTC"
    let sign = HmacSHA256(signStr, "fc27af70-8a02-4759-a72e-f51d9f0857b2");
        this.$http({
            url: this.$http.adornUrl('/wallet/addresses'),
            method: 'get',
            params: this.$http.adornParams({
            'coin_type': "BTC"
            },),
            headers:{
            'x-appkey':appkey,
            'x-sign':sign
            }
        }).then(({data}) => {
            if(data.code==0){
            //成功
            }else{
                //失败
            }
        })
}
~~~
成功返回：

~~~

{
    "msg":"success",
    "code":0,
    "data":{
        "address":"mvacHi6G1KxvwWy6VaY4Dj7LZAfM1AThUN",
        "coin_type":"BTC"
    }
}

~~~

## 2.API

**数据交互格式默认为:`application/json`**


需要说明的是：所有接口的出参格式都是一样的，如下：

~~~
{
  "msg":"success",
  "code":0,
   "data":{
    }
}
~~~
接口返回形式遵循  `https://jsonapi.org/` 标准。

即当接口请求成功的时候，返回`data`部分；同理，请求接口失败的`data `不返回，只返回"msg":"",
  "code":,部分。

#### 接口返回成功：

```
{
  "msg":"success",
  "code":0,
   "data":{
    }
}

```

#### 接口返回失败：

```
{
  "msg":"参数错误",
  "code":400
}

```




`msg`部分参数格式都是相同的，只有`data`部分各不相同。下面是`error`的参数格式：

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code       | string | 返回结果码 |
| msg    | string | 返回结果信息  |

错误码对照表见文档底部。

下面在介绍各个接口出参的参数格式的时候，默认是`data`的参数格式。



### API 请求范例

 获取一个新的地址:

```
curl -i -H 'Content-Type: application/json' -H 'x-appkey: 39dd949c-f748-4f9a-9620-2d7cb8f850a5'
-H 'x-sign: f9451e428f3f997d14fb1b2a5c9b0bafb0df1c77a2848959589d286b67442b1e' -XGET  https://www.morning.com/wallet/addresses -d '{ "coin_type": "BTC" }
```

成功返回：

~~~

{
    "msg":"success",
    "code":0,
    "data":{
        "address":"mvacHi6G1KxvwWy6VaY4Dj7LZAfM1AThUN",
        "coin_type":"BTC"
    }
}

~~~

如果API调用失败，返回的请求会使用对应的HTTP status code, 同时返回包含了详细错误信息的JSON数据, 比如:

~~~json
{"msg":"参数错误","code":400}
~~~

所有错误都遵循上面例子的格式，只是`code`和`msg`不同。


### API列表

获取新的地址
`POST /wallet/addresses`

获取单个账户余额
`GET /wallet/balance/{coin_type}{accounts}`

获取单个账户信息
`GET /wallet/accounts/{coin_type}{accounts}`

获取单个账户交易记录
`GET /wallet/list/{coin_type}{accounts}{page}{limit}`

获取充值记录
`GET /wallet/deposits/{coin_type}{accounts}{page}{limit}`

获取提现记录
`GET /wallet/withdraws/{coin_type}{accounts}{page}{limit}`

热钱包提现请求
`POST /wallet/transfer/{coin_type}{to}{quantity}{memo}`

获取通过序列号获取提现记录（验证提现记录是否在saas平台存在）
`GET /wallet/getTransfer/{coin_type}{serial_number}`

#### `POST /wallet/addresses `  获取新的地址

* URL 

`{{TEST_URL}}/wallet/addresses`
let signStr = "POST|/wallet/addresses|BTC"

* 入参
 
| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| address        | string | 地址    |

* 请求示例

```json
# Request
POST {{TEST_URL}}/wallet/addresses


curl -i -H 'Content-Type: application/json' -H 'x-appkey: 39dd949c-f748-4f9a-9620-2d7cb8f850a5' -H 'x-sign: 64ad8958074e7ebb9ede54bb0d002f29896dec8f0211d22566504a63a1daf8ef' -XPOST  https://www.morning.com/wallet/addresses -d '{ "coin_type": "BTC" }

# Response
{
    "msg":"success",
    "code":0,
    "data":{
        "address":"n1KqgwyWuqTdBQDpoPwXuDywBx4X6VGTEU",
        "coin_type":"BTC"
    }
}

```

#### `GET /wallet/addresses `  获取新的地址

* URL 

`{{TEST_URL}}/wallet/addresses`

let signStr = "GET|/wallet/addresses|BTC"

* 入参
 
| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| address        | string | 地址    |

* 请求示例

```json
# Request
GET {{TEST_URL}}/wallet/addresses


curl -i -H 'Content-Type: application/json' -H 'x-appkey: 39dd949c-f748-4f9a-9620-2d7cb8f850a5'
-H 'x-sign: f9451e428f3f997d14fb1b2a5c9b0bafb0df1c77a2848959589d286b67442b1e' -XGET  https://www.morning.com/wallet/addresses -d '{ "coin_type": "BTC" }

# Response
{
    "msg":"success",
    "code":0,
    "data":{
        "address":"n1KqgwyWuqTdBQDpoPwXuDywBx4X6VGTEU",
        "coin_type":"BTC"
    }
}

```
####  `GET /wallet/accounts/{coin_type}{accounts} ` 获取账户详情
/wallet/accounts/{coin_type}{accounts}`
 注意：调用此接口之前，请先调用地址验证接口以保证地址（accounts）可用

* URL 

`{{TEST_URL}}/wallet/accounts/{coin_type}{accounts}`
let signStr = "GET|/wallet/accounts|EOS|vbuspyjtst3p"
* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| accounts        | string | 账号    |



* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |



* 请求示例

```json
# Request
POST {{TEST_URL}}/wallet/withdraws

# Response
{
    "msg":"success",
    "code":0,
    "data":{
        "creator":"ge2tgnztgige",
        "create_timestamp":"2018-09-30T12:48:36.000",
        "permissions":[
            {
                "parent":"owner",
                "required_auth":{
                    "waits":[

                    ],
                    "keys":[
                        {
                            "weight":1,
                            "key":"EOS7AyX3pZ62XPh35Fs2qLSm7JQ2C9UcuqP1RGSHUjipbexnvFiHR"
                        }
                    ],
                    "threshold":1,
                    "accounts":[

                    ]
                },
                "perm_name":"active"
            },
            {
                "parent":"",
                "required_auth":{
                    "waits":[

                    ],
                    "keys":[
                        {
                            "weight":1,
                            "key":"EOS5o6EHEYjxh11CfNsmeWWVxRVg6ks7oK6r75HdK29Ppwn6rG2rm"
                        }
                    ],
                    "threshold":1,
                    "accounts":[

                    ]
                },
                "perm_name":"owner"
            }
        ]
    }
}
```
####  `GET /wallet/balance/{coin_type}{accounts} ` 获取账户余额
/wallet/balance/{coin_type}{accounts}`
 注意：调用此接口之前，请先调用地址验证接口以保证地址（accounts）可用

* URL 

`{{TEST_URL}}/wallet/balance/{coin_type}{accounts}`

let signStr = "GET|/wallet/balance|EOS|vbuspyjtst3p"

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| accounts        | string | 账号    |



* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |


* 请求示例

```json
# Request
`{{TEST_URL}}/wallet/balance/{coin_type}{accounts}`

# Response
{
    "msg":"success",
    "code":0,
    "data":{
        "id":1,
        "account":"kpd1uhb1ulsj",
        "balance":5.0241,
        "stakeToOthers":0,
        "stakeToSelf":0.2,
        "unstake":0
    }
}
```

#### `GET /wallet/list/{coin_type}{accounts}{page}{limit}`  获取单个账户交易记录

* URL 

`{{TEST_URL}}/wallet/list/{coin_type}{accounts}{page}{limit}`

let signStr = "GET|/wallet/list|EOS|vbuspyjtst3p"
* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| accounts  | string | 币种类型 |
| page  | int | 页参数 （从1开始） |
| limit  | int | 每页数量（每页最大值20） |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |

* 请求示例

```json
# Request
GET {{TEST_URL}}/wallet/list/{coin_type}{accounts}{page}{limit}

# Response
{
    "msg":"success",
    "code":0,
    "page":{
        "totalCount":140,
        "pageSize":1,
        "totalPage":140,
        "currPage":1,
        "list":[
            {
                "id":2785,
                "userId":1545828668984,
                "status":"new",
                "trxId":"58c90e36020ef2b99dfd4d8d3908dd810f4f6ee901d0f4d27019cef28d9a1f34",
                "isCallback":"3",
                "account":"kpd1uhb1ulsj",
                "tokenName":"EOS",
                "transferFrom":"fisimtoken4y",
                "transferTo":"kpd1uhb1ulsj",
                "quantity":0.0001,
                "memo":"Perfect ecological DAPP Perfect DAPP for blockchain thinking. Download now: http://fis.im",
                "createdat":"2019-01-06 07:24:21",
                "createtime":"2019-01-08 21:07:00",
                "updatetime":"2019-01-08 21:07:00",
                "blockNum":35971959
            }
        ]
    }
}
```

获取充值记录
`GET /wallet/deposits/{coin_type}{accounts}{page}{limit}`
#### `GET /wallet/deposits/{coin_type}{accounts}{page}{limit}`  获取充值记录

* URL 

`{{TEST_URL}}/wallet/deposits/{coin_type}{accounts}{page}{limit}`
let signStr = "GET|/wallet/deposits|EOS|vbuspyjtst3p"
* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| accounts  | string | 币种类型 |
| page  | int | 页参数 （从1开始） |
| limit  | int | 每页数量（每页最大值20） |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |

* 请求示例

```json
# Request
GET {{TEST_URL}}/wallet/deposits/{coin_type}{accounts}{page}{limit}

# Response
{
    "msg":"success",
    "code":0,
    "page":{
        "totalCount":60,
        "pageSize":1,
        "totalPage":60,
        "currPage":1,
        "list":[
            {
                "id":2785,
                "userId":1545828668984,
                "status":"new",
                "trxId":"58c90e36020ef2b99dfd4d8d3908dd810f4f6ee901d0f4d27019cef28d9a1f341",
                "isCallback":"3",
                "account":"kpd1uhb1ulsj",
                "tokenName":"EOS",
                "transferFrom":"xxxxxxx",
                "transferTo":"kpd1uhb1ulsj",
                "quantity":0.0001,
                "memo":"Perfect ecological now: h",
                "createdat":"2019-01-06 07:24:21",
                "createtime":"2019-01-08 21:07:00",
                "updatetime":"2019-01-08 21:07:00",
                "blockNum":35971959
            }
        ]
    }
}
```

获取提现记录
`GET /wallet/withdraws/{coin_type}{accounts}{page}{limit}`
#### `GET /wallet/withdraws/{coin_type}{accounts}{page}{limit}`  获取提现记录

* URL 

`{{TEST_URL}}/wallet/withdraws/{coin_type}{accounts}{page}{limit}`

let signStr = "GET|/wallet/withdraws|EOS|vbuspyjtst3p"

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| accounts  | string | 币种类型 |
| page  | int | 页参数 （从1开始） |
| limit  | int | 每页数量（每页最大值20） |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |

* 请求示例

```json
# Request
GET {{TEST_URL}}/wallet/withdraws/{coin_type}{accounts}{page}{limit}

# Response
{
    "msg":"success",
    "code":0,
    "page":{
        "totalCount":80,
        "pageSize":1,
        "totalPage":80,
        "currPage":1,
        "list":[
            {
                "id":2802,
                "userId":1545828668984,
                "status":"new",
                "trxId":"ba7879144f6e9479fd5735d6b503e4b6d64d04d3f6d4a92d0a531e8a7b867d6a",
                "isCallback":"3",
                "account":"kpd1uhb1ulsj",
                "tokenName":"EOS",
                "transferFrom":"kpd1uhb1ulsj",
                "transferTo":"xxxxx",
                "quantity":1.5539,
                "memo":"{"type":"buy-limit","symbol":"DICE_EOS","price":"0.000100","count":"15539.3700","amount":1.5539,"channel":"eostoken"}",
                "createdat":"2018-11-26 02:46:44",
                "createtime":"2019-01-08 21:07:01",
                "updatetime":"2019-01-08 21:07:01",
                "blockNum":28887293
            }
        ]
    }
}
```

热钱包提现请求
`POST /wallet/transfer/{coin_type}{to}{quantity}{memo}{serial_number}{sign}`
#### `POST /wallet/transfer/{coin_type}{to}{quantity}{memo}{serial_number}{sign}`  热钱包提现请求

* URL 

`{{TEST_URL}}/wallet/transfer/{coin_type}{to}{quantity}{memo}{serial_number}`

let signStr = "POST|/wallet/transfer|"+"EOS|kpd1uhb1ulsj|0.0001|Mark|1d37374b351a0d608a79c1230eefc80cc1aa41a5429e5c5944bb14218a6c8c78"

加密步骤说明：采用RSA私钥加密，公钥解密方案,用户下载RSATOOL.java 
1、生成公钥和私钥。
2、申请开通服务权限。
3、登录后台管理系统，在系统管理子菜单的热钱包配置中配置（热包提现通信公钥）。
4、byte[] signBytes =encryptByRSA1(priKey ,signStr);
5、String signRsa = byteToStr(signBytes)
6、在header里面添加x-rsa-sign :signRsa
 

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| to  | string | 目标账号 |
| quantity  | string | 转账金额 |
| memo  | string | 备注 |
| serial_number  | string | 生成唯一id（UUID），用于验证此条记录是否成功推送到saas平台。场景：由于外界原因未收到saas的成功返回，可用于验证。 |
| sign  | string | RSA签名 （不需要进行加x-sign加密）|

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |


* 请求示例

```json
# Request
`{{TEST_URL}}/wallet/transfer/{coin_type}{to}{quantity}{memo}{serial_number}{sign}`

# Response
{
    "msg":"success",
    "code":0,
}
      
```

获取通过序列号获取提现记录（验证提现记录是否在saas平台存在）
`GET /wallet/getTransfer/{coin_type}{serial_number}`
#### `GET /wallet/getTransfer/{coin_type}{serial_number}`  获取通过序列号获取提现记录

* URL 

`{{TEST_URL}}/wallet/getTransfer/{coin_type}{serial_number}`

let signStr = "GET|/wallet/getTransfer|EOS|55dd949c-f748-4f9a-9620-2d7cb8f850a5"

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| coin_type  | string | 币种类型 |
| serial_number  | string | 币种类型 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code  | int |返回码 |
| msg  | string | 返回消息 |
| data  | JSON | 详情 |

* 请求示例

```json
# Request
GET {{TEST_URL}}/wallet/getTransfer/{coin_type}{serial_number}

# Response
{
    "msg":"success",
    "code":0,
    "data":{
        "id":69,
        "userId":1545828668984,
        "eosid":"l2dzsphvirge",
        "memo":"",
        "createtime":"2019-01-14 16:03:59",
        "qty":0.0001,
        "cointype":"EOS",
        "updatetime":"2019-01-14 19:41:59",
        "status":"ok",
        "txid":"d3a00c21c91bf2df124751634fb5d41903147a679ecfd931de9cd1269f3fb92a",
        "direction":-1,
        "target":"kpd1uhb1ulsj",
        "oprationId":null,
        "sign":"1d37374b351a0d608a79c1230eefc80cc1aa41a5429e5c5944bb14218a6c8c78",
        "serialNumber":"55dd949c-f748-4f9a-9620-2d7cb8f850a5"
    }
}
```




## 3.业务场景流程分析 



## 错误码对照表

| 错误码       |       意义         | 中文解释                   |
| ----------- | ------------------ |  --------                  |
| 0        | 请求成功 | 成功 |
| 403        | 没有权限 |  没有权限    |
| 400        | 参数错误 |  必要参数不能为空。           |
| 3000        | x-appkey  error.  |  x-appkey  参数错误       |
| 3001        | x-sign签名不一致 | x-sign参数错误             |
| 3002        | coin_type参数错误 |  coin_type参数错误       |
| 3003        | x-appkey 参数错误，用户不存在       | x-appkey 参数错误，用户不存在 |
| 3004        | x-sign签名失败 |x-sign签名失败|
| 3005        | 冷钱包未配置 |  冷钱包未配置             |
| 3006        | 未开通服务或服务到期,请充值。 | 未开通服务或服务到期,请充值。             |
| 3007        | coin_type 错误 |  coin_type 错误            |
| 3008       | 账户不存在，或未开通！ |  账户不存在，或未开通！            |
| 3009        | 请配置热钱包安全配置 | 请配置热钱包安全配置        |
| 3010        | 转账签名不一致 |转账签名不一致      |
| 3011        | 生成地址失败|  生成地址失败             |

