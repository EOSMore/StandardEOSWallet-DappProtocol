# StandardEOSWallet Protocol Standard

Version：1.0.0

Latest Update：20180817

Support Platform：WEB

[中文版](https://github.com/EOSMore/StandardEOSWallet-DappProtocol/blob/master/Readme-zh.md)

## Date Format

### QR Code Format

Use `json` format，field as below：

|   Data    |  Type   |                            State                             |
| :-------: | :-----: | :----------------------------------------------------------: |
| protocol  | string  |        protocol name, assignment is StandardEOSWallet        |
|  version  | string  |                   protocol version number                    |
|  action   | string  |                         action type                          |
| action_id | string  |                specific ID of current action                 |
|  expired  | integer |           action expired timestamp, unit is second           |
|   data    | object  |                         action data                          |
|   memo    | string  |                   request memo information                   |
| callback  | string  |                action callback integrate link                |
|   dapp    | object  | includes two data：name and icon，they are dapp name and logo link |

### Callback Integrate Public Request Data

> All Data requested for calling all callback integration

|   Data   |  Type |                State                |
| :-------: | :-----: | :--------------------------------: |
| protocol  | string  | protocol name, assignment is StandardEOSWallet |
|  version  | string  |             protocol version number             |
|  action   | string  |              action type              |
| action_id | string  |           specific ID of current action           |
|  account  | string  | action eos account     |
| timestamp | integer |         action timestamp, unit is second         |
|  source   | string  |     name of source wallet，such as MoreWallet     |
|   sign    | string  | generic sign       |

### Callback Interface Return Data

|  Data   |  Type   |                      State                      |
| :-----: | :-----: | :---------------------------------------------: |
|  code   | integer | code=0 means success, other values mean failure |
| message | string  |                  error message                  |

## Login

### QR Code Data Example

```json
{
    "protocol": "StandardEOSWallet",
    "version": "1.0.0",
    "action": "login",
    "action_id": "xxxxxx",
    "expired": 1534347099,
    "data": {},
    "memo": "This is a xxx eos dapp",
    "callback": "https://api.dappdemo.com/login",
    "dapp": {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

### Sign

```javascript
// generate sign algorithm
let data = timestamp + account + action_id + source // source is wallet name, indicate the source
sign = ecc.sign(data, privateKey)
```

### Callback Interface Request Data

| Data |  Type  |              State              |
| :--: | :----: | :-----------------------------: |
| sign | string | Sign accquired in previous step |

## Transfer

### QR Code Data Example

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
        "memo": "transfer remark"
    },
    "memo": "use for renting cpu",
    "callback": "https://api.dappdemo.com/transfer/callback",
    "dapp": {
        "name": "Southex",
        "icon": "http://swdemo.southex.com/static/southex.png"
    }
}
```

### Callback Interface Request Data

|      Data      |  Type  |     State      |
| :------------: | :----: | :------------: |
| transaction_id | string | transaction ID |

## Submit Transaction

### QR Code Data Example

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

### Callback Interface Request Data

|      Data      |  Type  |     State      |
| :------------: | :----: | :------------: |
| transaction_id | string | Transaction ID |

## Paging

### Paging example when QR code content overlong

**Overlong json content will lead to the QR code unidentified when several Actions existed. Details will be setting pageable as total pages in field data, APP will enter continuous scanning mode when identified pageable existed, paging content should be designed as below**

**Open continuous scanning mode when identified pageable, and estimate if following scan content belong to continuous scanning mode account to action_id**

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

**Estimate if the content belongs to current continuous scanning mode according to actioin_id, and cover same content repeat scanning by page field**

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

**Continuous scanning will estimate if current stored actions length equal to pageable every ended, stop scanning if yes**

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


***If nonexistent or different action_id content scanned under continuous scanning mode, suggest to quit the continuous scanning mode first and identify again***