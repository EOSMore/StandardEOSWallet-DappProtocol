# StandardEOSWallet 协议标准

版本：1.0.0

最后更新：20180817

支持平台：WEB端

## 数据格式

### 二维码数据格式

采用`json`格式，字段如下：

|   参数    |  类型   |                           说明                           |
| :-------: | :-----: | :------------------------------------------------------: |
| protocol  | string  |            协议名称，赋值为 StandardEOSWallet            |
|  version  | string  |                        协议版本号                        |
|  action   | string  |                         操作类型                         |
| action_id | string  |                      本次操作唯一ID                      |
|  expired  | integer |                  操作过期时间戳，单位秒                  |
|   data    | object  |                         操作参数                         |
|   memo    | string  |                       请求说明信息                       |
| callback  | string  |                     操作回调接口链接                     |
|   dapp    | object  | 包含两个参数：name 和 icon，分别为 dapp 名称和 logo 链接 |

### 回调接口公共请求参数

> 调用所有回调接口都需要传这些参数

|   参数    |  类型   |                说明                |
| :-------: | :-----: | :--------------------------------: |
| protocol  | string  | 协议名称，赋值为 StandardEOSWallet |
|  version  | string  |             协议版本号             |
|  action   | string  |              操作类型              |
| action_id | string  |           本次操作唯一ID           |
|  account  | string  |            操作eos账号             |
| timestamp | integer |         操作时间戳，单位秒         |
|  source   | string  |     来源钱包名称，如MoreWallet     |
|   sign    | string  |              通用签名             |

### 回调接口返回参数

|  参数   |  类型   |             说明              |
| :-----: | :-----: | :---------------------------: |
|  code   | integer | code=0表示成功 其他值表示失败 |
| message | string  |           错误信息            |

## 登录

### 二维码数据示例

```json
{
    "protocol": "StandardEOSWallet",
    "version": "1.0.0",
    "action": "login",
    "action_id": "xxxxxx",
    "expired": 1534347099,
    "data": {},
    "memo": "这是一款xxx的eos dapp",
    "callback": "https://api.dappdemo.com/login",
    "dapp": {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

### 签名

```javascript
// 生成sign算法
let data = timestamp + account + action_id + source // source为钱包名，标示来源
sign = ecc.sign(data, privateKey)
```

### 回调接口请求参数

| 参数 |  类型  |       说明       |
| :--: | :----: | :--------------: |
| sign | string | 上一步获取的签名 |

## 转账

### 二维码数据示例

```json
{
    "protocol": "StandardEOSWallet",
    "version": "1.0.0",
    "action": "transfer",
    "action_id": "xxxxxx",
    "expired": 1534347099,
    "data": {
        "contract": "eosio.token",
        "from": "loginaccount",
    	"to": "transferto22",
        "quantity": "10.0000 EOS",
        "memo": "转账备注"
    },
    "memo": "用于租赁cpu",
    "callback": "https://api.dappdemo.com/transfer/callback",
    "dapp": {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

### 回调接口请求参数

|      参数      |  类型  |  说明  |
| :------------: | :----: | :----: |
| transaction_id | string | 交易ID |

## 提交transaction

### 二维码数据示例

```json
{
    "protocol": "StandardEOSWallet",
    "version": "1.0.0",
    "action": "push_transaction",
    "action_id": "xxxxxx",
    "expired": 1534347099,
    "data": {
        "actions": [{
            "account": "eosio",
            "name": "buyram",
            "authorization": [{
                "actor": "demouser1111",
            	"permission": "active"
            }],
            "data": {
                "payer": "demouser1111",
            	"receiver": "demouser1111",
            	"quant": "10.0000 EOS"
          	}
        }, {
            "account": "eosio",
            "name": "delegatebw",
            "authorization": [{
            	"actor": "demouser1111",
            	"permission": "active"
            }],
            "data": {
            	"from": "demouser1111",
            	"receiver": "demouser1111",
            	"stake_net_quantity": "10.0000 EOS",
            	"stake_cpu_quantity": "5.0000 EOS",
            	"transfer": false
            }
        }]
    },
    "memo": "",
    "callback": "https://api.dappdemo.com/push_transaction/callback",
    "dapp": {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

### 回调接口请求参数

|      参数      |  类型  |  说明  |
| :------------: | :----: | :----: |
| transaction_id | string | 交易ID |

## 分页

### 当二维码内容过长时, 分页示例

**当存在多个Actions可能会因为json内容过长导致二维码无法识别, 建议使用分页模式, 具体则是通过在字段data中设置pageable的为总页数, APP识别到pageable的存在则会进入连续扫码模式, 分页内容应如下设计**


**检测到pageble开启连续扫码模式, 并根据action_id判断接下来的扫码内容是否为当前连续扫码模式**
```json
{
    "protocol": "StandardEOSWallet",
    "version": "1.0.0",
    "action": "push_transaction",
    "action_id": "xxxxxx",
    "expired": 1534347099,
    "data":
    {
        "pageable": 2
    },
    "memo": "",
    "callback": "https://api.dappdemo.com/push_transaction/callback",
    "dapp":
    {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

**根据action_id判断是属于当前连续模式下内容, 并通过page字段来覆盖相同内容的重复扫描**
```json

{
    "action_id": "xxxxxx",
    "data":
    {
        "page": 1,
        "actions": [
        {
            "account": "eosio",
            "name": "buyram",
            "authorization": [
            {
                "actor": "demouser1111",
                "permission": "active"
            }],
            "data":
            {
                "payer": "demouser1111",
                "receiver": "demouser1111",
                "quant": "10.0000 EOS"
            }
        }]
    }
}
```

**连续扫码每次结束后判断当前存储的actions lenght是否等于pageable, 若是则结束扫码**
```json
{
    "action_id": "xxxxxx",
    "data":
    {
        "page": 2,
        "actions": [
        {
            "account": "eosio",
            "name": "delegatebw",
            "authorization": [{
                "actor": "demouser1111",
                "permission": "active"
            }],
            "data": {
                "from": "demouser1111",
                "receiver": "demouser1111",
                "stake_net_quantity": "10.0000 EOS",
                "stake_cpu_quantity": "5.0000 EOS",
                "transfer": false
            }
        }]
    }
}
```

***若在连续扫码模式下, 扫描到不存在action_id或action_id不相同的内容, 建议退出连续扫码模式, 重新识别内容***

