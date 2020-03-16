# RPC概览

## JSON-RPC methods

- [Gch_tx]()
- [Gch_nonce]()
- [Gch_txReceipt]()
- [Gch_balance]()
- [Gch_groupHeight]()
- [Gch_getBlockByHeight]()
- [Gch_getBlockByHash]()
- [Gch_getTxsByBlockHeight]()
- [Gch_getTxsByBlockHash]()
- [Gch_minerPoolInfo]()
- [Gch_minerInfo]()
- [Gch_transDetail]()
- [Gch_checkPointAt]()
- [Gch_latestCheckPoint]()



### Gch_tx [⇧](https://docs.chiron.one/api.html)

发送交易

#### Parameters

1. ```
   Object
   ```

   -transaction交易体，json格式

   - `source`: 发送交易源地址
   - `target`: 接收地址
   - `value`:发送的金额, 单位RA，各单位间换算：109RA=106kRA=103mRA=1Chc
   - `gas_limit`: 该交易可消耗gas的最大限制
   - `gas_price`: gas价格，默认0
   - `type`: 交易类型，普通交易为0
   - `nonce`: source 账户的nonce
   - `data`: 普通交易可不传该参数，参数类型为base64格式的字符串
   - `sign`: 签名, hex格式
   - `extra_data`: 可选备注， 参数类型为base64格式的字符串

#### Example Parameters

```
params: [{
    "source": "0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e",
    "target": "0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e", 
    "value": 1000000000, 
    "gas_limit": 1000,  
    "gas_price": 0,
    "type": 0,  
    "nonce": 10, 
    "data": "", 
    "sign": "0x32f4b12bfba23fbe64043becc239184f7aeccbc815f4771058907ab01379062f51f580ea494d5d70e3ae3326fc5dc90946e9629689dddab6ced86deaf3b911ea02", 
    "extra_data": ""  
}]
```

#### Returns

`Hash` - 所发送交易的hash

#### Example

```
# Request
import json
import requests

url = 'http://39.101.192.249:8101/'

transaction = {
    "source": "0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e", 
    "target": "0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e", 
    "value": 1000000000,  
    "gas_limit": 1000,  
    "gas_price": 0, 
    "type": 0,  
    "nonce": 10, 
    "data": "",  
    "sign": "0x32f4b12bfba23fbe04043becc339184f7aeccbc805f4771058907ab01379062f51f580ea494d5d70e3ae3326fc5dc90946e9d29689dddab6ced76deaf3b911ea02",  # 签名, hex格式
    "extra_data": "" 
}

payload = {
    "method": "Gch_tx",
    "params": [
        transaction
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc":"2.0",
    "id":"1",
    "result":"0xf86fcfd8c1fb2572d831fc50e88da87311d1a197c6eb6133e7eddcc788f06e0d"
}
```

### Gch_nonce [⇧]()

获取指定账户的nonce

#### Parameters

1. `Address`-查询的地址

#### Example Parameters

```
"params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf"
 ]
```

#### Returns

`uint`-指定账户的nonce值

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_nonce",
    "params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": 1 
}
```

### Gch_txReceipt [⇧]()

查询交易回执

#### Parameters

`Hash`: 交易hash

#### Example Parameters

```
"params": [
        "0xf86fcfd8c1fb2572d831fc50e88da87311d1a197c6eb6133e7eddcc788f06e0d"
 ]
```

#### Returns

`Object`-回执和交易数据

- ```
  Receipt
  ```

  - `status`-回执状态码
  - `cumulativeGasUsed`-交易gas消耗
  - `logs`-合约产生的log
  - `transactionHash`-交易hash
  - `contractAddress`-如果交易类型为创建合约，显示创建的合约地址
  - `height`-交易所在块高
  - `tx_index`-交易在块中的索引

- ```
  Transaction
  ```

  - `data`-交易data数据
  - `value`-交易的转账数额，单位
  - `nonce`-交易的nonce
  - `source`-发送交易源地址
  - `target`-交易接收地址
  - `type`-交易类型
  - `gas_limit`-交易可消耗gas
  - `gas_price`-gas价格，单位
  - `hash`-交易hash
  - `extra_data`-备注信息

#### Example

```
# Request
import json
import requests

url = 'http://39.101.192.249:8101/'

payload = {
    "method": "Gch_txReceipt",
    "params": [
        "0xf86fcfd8c1fb2572d831fc50e88da87311d1a197c6eb6133e7eddcc788f06e0d"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
/*
回执状态码
RSSuccess ReceiptStatus = 0 // 交易成功
RSFail= 1                   // 交易失败
RSBalanceNotEnough= 2				// 余额不足
RSAbiError= 3								// 执行合约abi校验失败
RSTvmError= 4								// 执行合约失败
RSGasNotEnoughError= 5			// gas不足
RSNoCodeError= 6						// 请求地址无合约代码
RSParseFail= 7							// 解析错误
RSMinerStakeFrozen= 8 //矿工处于冻结状态不允许操作
RSMinerStakeOverLimit= 9 //质押超过上限不允许增加质押
RSMinerStakeLessThanReduce= 10 //减少质押数量大于用户质押数量
RSMinerVerifyLowerStake= 11 //矿工正常状态下，验证节点质押数量减少到门槛500以下不允许操作
RSMinerVerifyInGroup= 12 //矿工有有效验证组，不允许减少到门槛500以下
RSMinerReduceHeightNotEnough= 13 //未达到可允许减少块高不允许减质押
RSVoteNotInRound= 14 //当前周期已经投过票了
RSMinerUnSupportOp= 15 //不支持的操作
RSMinerNotFullStake= 16 //未满质押不允许申请成为守护节点
RSMinerMaxApplyGuard= 17 //守护节点最大申请块高超过限制
RSMinerChangeModeExpired= 18 //改守护节点模式时间已过，不允许操作
RSMinerAbortHasPrepared= 19 //矿工状态已处于暂停挖矿状态
RSMinerRefundHeightNotEnough= 20 //可允许退减少质押的币至账号的块高还未到，不允许操作
RSMinerNotExists
RSMinerActiveAlready 21//矿工已激活
*/
{
    "jsonrpc":"2.0",
    "id":"1",
    "result":{
        "Receipt":{
            "status":0, 
            "cumulativeGasUsed":400, 
            "logs":null,  
            "transactionHash":"0xf86fcfd8c1fb2572d831fc50e88da87311d1a197c6eb6133e7eddcc788f06e0d",
            "contractAddress":"0x0000000000000000000000000000000000000000000000000000000000000000",
            "height":6830, 
            "tx_index":1
        },
        "Transaction":{
            "data":null,
            "value":1,
            "nonce":7,
            "source":"0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e",
            "target":"0xd4d108ca92871ab1115439db841553a4ec3c8eddd955ea6df299467fbfd0415e",
            "type":0,
            "gas_limit":400,
            "gas_price":500,
            "hash":"0xf86fcfd8c1fb2572d831fc50e88da87311d1a197c6eb6133e7eddcc788f06e0d",
            "extra_data":""
        }
    }
}
```

### Gch_balance [⇧]()

获取指定账户的余额

#### Parameters

1. `Address`-查询的地址

#### Example Parameters

```
"params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf"
 ]
```

#### Returns

`float`-指定账户的余额(单位：0xC)

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_balance",
    "params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": 1.1 
}
```

### Gch_blockHeight [⇧]()

获取节点当前同步块高

#### Parameters

```
none
```

#### Example Parameters

```
"params": [
 ]
```

#### Returns

`uint`-节点当前同步块高

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_blockHeight",
    "params": [
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": 1024 
}
```

### Gch_groupHeight [⇧]()

获取节点当前同步组高

#### Parameters

```
none
```

#### Example Parameters

```
"params": [
 ]
```

#### Returns

`uint`-节点当前同步组高

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_groupHeight",
    "params": [
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": 1024
}
```

### Gch_getBlockByHeight [⇧]()

获取指定块高查询块信息

#### Parameters

1. `uint`-查询的块高

#### Example Parameters

```
"params": [
        1024
 ]
```

#### Returns

`Obejct`-块信息

- `height`-区块高度
- `hash`-当前区块hash
- `pre_hash`-上一块的区块hash
- `cur_time`-当前块时间
- `pre_time`-上一块时间
- `castor`-提案者地址
- `group_id`-验证组id
- `prove`
- `total_qn`-总块权重
- `qn`-当前块权重
- `txs`-块所包含交易数
- `state_root`-state_db的根hash
- `tx_root`-交易根hash
- `receipt_root`-回执根hash
- `prove_root`
- `random`-随机值

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_getBlockByHeight",
    "params": [
        1024
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "height": 1024,
        "hash": "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d",
        "pre_hash": "0x63effc19ac273e3cccfd84e26cae531eddf0197258f37d25ee3e9d92ff979899",
        "cur_time": "2019-09-03T16:25:10.763+08:00",
        "pre_time": "2019-09-03T16:25:07.763+08:00",
        "castor": "0x938407bee35f42fe07bca44a0463088bbfbf19d821212ad4b2a1da284127d79a",
        "group_id": "0x6861736820666f72207a76636861696e27732067656e657369732067726f7570",
        "prove": "0x03c4d77a9976bbcfc6af62e5abd2d129c8ca643611359795ecad3127486c1927dd5fa7498f434d3262253961df4a5845670321f18a1bc68709182b28054034bf201d635177777fd78db80fec91aeb22527",
        "total_qn": 4984,
        "qn": 5,
        "txs": 0,
        "state_root": "0xb4b4ed55abcb67ac96b5e051575a5e038ab6db16ef0a2ed9f425f716b025acb3",
        "tx_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "receipt_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "prove_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "random": "0x08fe8d72bfddd1a52173671c3a0c95e8c93564e694723a48fd0004046866d61400"
    }
}
```

### Gch_getBlockByHash [⇧]()

获取指定账户的余额

#### Parameters

1. `Hash`-查询的块的hash值

#### Example Parameters

```
"params": [
        "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
 ]
```

#### Returns

- ```
  Obejct
  ```

  -块信息

  - `height`-区块高度
  - `hash`-当前区块hash
  - `pre_hash`-上一块的区块hash
  - `cur_time`-当前块时间
  - `pre_time`-上一块时间
  - `castor`-提案者地址
  - `group_id`-验证组id
  - `prove`
  - `total_qn`-总块权重
  - `qn`-当前块权重
  - `txs`-块所包含交易数
  - `state_root`-state_db的根hash
  - `tx_root`-交易根hash
  - `receipt_root`-回执根hash
  - `prove_root`
  - `random`-随机值

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_getBlockByHash",
    "params": [
        "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "height": 1024,
        "hash": "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d",
        "pre_hash": "0x63effc19ac273e3cccfd84e26cae531eddf0197258f37d25ee3e9d92ff979899",
        "cur_time": "2019-09-03T16:25:10.763+08:00",
        "pre_time": "2019-09-03T16:25:07.763+08:00",
        "castor": "0x938407bee35f42fe07bca44a0463088bbfbf19d821212ad4b2a1da284127d79a",
        "group_id": "0x6861736820666f72207a76636861696e27732067656e657369732067726f7570",
        "prove": "0x03c4d77a9976bbcfc6af62e5abd2d129c8ca643611359795ecad3127486c1927dd5fa7498f434d3262253961df4a5845670321f18a1bc68709182b28054034bf201d635177777fd78db80fec91aeb22527",
        "total_qn": 4984,
        "qn": 5,
        "txs": 0,
        "state_root": "0xb4b4ed55abcb67ac96b5e051575a5e038ab6db16ef0a2ed9f425f716b025acb3",
        "tx_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "receipt_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "prove_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "random": "0x08fe8d72bfddd1a52173671c3a0c95e8c93564e694723a48fd0004046866d61400"
    }
}
```

### Gch_getTxsByBlockHash [⇧]()

获取指定块的所有交易Hash

#### Parameters

1. `Hash`-查询的块的hash值

#### Example Parameters

```
"params": [
        "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
 ]
```

#### Returns

`Array`-交易Hash

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_getTxsByBlockHash",
    "params": [
        "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
      "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
    ]
}
```

### Gch_getTxsByBlockHeight [⇧]()

获取指定块的所有交易Hash

#### Parameters

1. `uint`-查询的块的高度

#### Example Parameters

```
"params": [
        1024
 ]
```

#### Returns

`Array`-交易Hash

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_getTxsByBlockHeight",
    "params": [
        1024
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
      "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
    ]
}
```

### Gch_minerPoolInfo [⇧]()

查询某高度的矿工信息

#### Parameters

1. `Address`-查询的地址
2. `uint`-查询高度

#### Example Parameters

```
"params": [
        "0x61dab936abb18d68c32fab6ed9b0eeb42d80d3e2b982a2e765864e73588f7805", 100
 ]
```

#### Returns

`Object`-矿工信息

- `current_stake`-当前质押，单位RA
- `full_stake`-满质押金额，单位RA，仅矿池该值有效
- `tickets`-被投票数
- `identity`-身份，0正常矿工，1守护节点，2矿池节点，3失效矿池节点
- `valid_tickets`-有效票数

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_minerPoolInfo",
    "params": [
        "0x61dab936abb18d68c32fab6ed9b0eeb42d80d3e2b982a2e765864e73588f7805", 100
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "current_stake": 1000000000000,
        "full_stake": 0,
        "tickets": 0,
        "identity": 0,
        "valid_tickets": 8
    }
}
```

### Gch_minerInfo [⇧]()

获取当前矿工质押信息

#### Parameters

1. `Address`-查询的被质押的地址
2. `Address|none`-质押的地址，空字符串表示被质押的地址

#### Example Parameters

```
"params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf", ""
 ]
```

#### Returns

`Object`-质押相关信息

- ```
  overview
  ```

  \- 总览

  - ```
    Array<Object>
    ```

    - ```
      Object
      ```

      -详情

      - `stake`-总质押金额，单位0xC
      - `apply_height`-申请高度
      - `type`-申请类型
      - `miner_status`-矿工状态,prepared停止挖矿状态,normal正常挖矿状态,frozen矿工账号冻结状态
      - `status_update_height`状态更新高度
      - `identity`-身份
      - `identity_update_height`-身份更新高度

- ```
  details
  ```

  \- 详情

  - ```
    Map<string, Array<Object>>
    ```

    -key值为质押人

    - Object`-详情
      - `value`-质押金额，单位0xC
      - `update_height`-质押更新高度
      - `m_type`-质押类型
      - `stake_status`-质押状态,normal正常的质押,frozen冻结的质押（减少的质押正常回退到账号内之前存储在此）,punish被惩罚的质押
      - `can_reduce_height`-可减少质押的块高

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_minerInfo",
    "params": [
         "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf", ""
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "overview": [
            {
                "stake": 1000,
                "apply_height": 0,
                "type": "proposal node",
                "miner_status": "normal",
                "status_update_height": 0,
                "identity": "normal node",
                "identity_update_height": 0
            },
            {
                "stake": 500,
                "apply_height": 0,
                "type": "verify node",
                "miner_status": "normal",
                "status_update_height": 0,
                "identity": "normal node",
                "identity_update_height": 0
            }
        ],
        "details": {
            "0x61dab936abb18d68c32fab6ed9b0eeb42d80d3e2b982a2e765864e73588f7805": [
                {
                    "value": 500,
                    "update_height": 0,
                    "m_type": "verifier",
                    "stake_status": "normal",
                    "can_reduce_height": 0
                },
                {
                    "value": 1000,
                    "update_height": 1,
                    "m_type": "proposal",
                    "stake_status": "normal",
                    "can_reduce_height": 0
                }
            ]
        }
    }
}
```

### Gch_transDetail [⇧]()

根据交易hash查询交易详情

#### Parameters

1. `Hash`-查询的交易hash

#### Example Parameters

```
"params": [
        "0x211e7a402a18f71bf77397db651f03b9094037de2769dce3567c991878723e4d"
 ]
```

#### Returns

`Obeject`-交易详情

- `data`- 交易的data区, base64编码
- `value`-交易金额, 0xC单位
- `nonce`-交易nonce
- `source`-交易发起者
- `target`-交易接受者
- `type`-交易类型
- `gas_limit`-交易可用gas
- `gas_price`-交易gas价格，单位RA
- `hash`-交易hash
- `extra_data`-交易附加内容

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_transDetail",
    "params": [
        "0x331db0f9e6ccee2dff9361cea599fbf46e528d0388c5223dc4ed7e0fd6c0323b"
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "data": "6sA6vI3l2d6Ymmo2rWEptqGNoJEny3D4pxmuaPNJzxc=",
        "value": 0.95706438,
        "nonce": 0,
        "source": null,
        "target": null,
        "type": 104,
        "gas_limit": 0,
        "gas_price": 0,
        "hash": "0x331db0f9e6ccee2dff9361cea599fbf46e528d0388c5223dc4ed7e0fd6c0323b",
        "extra_data": "\u0001\u007f\ufffd\u0002\u0002\ufffd/@\ufffd\u000b\ufffdaw\ufffd\u0002%\ufffd}\ufffd8\"\ufffd%\u0016\f\ufffdj\ufffd\ufffd\ufffd\ufffd\ufffd\u0019\u0000\u0000\u0000\u0001VE\ufffdm\u0000\u0000\u0000\u0001\u0000\u0003\u0000\u0005\u0000\u0007\u0000\t"
    } 
}
```

### Gch_queryAccountData [⇧]()

获取合约存储区的数据

#### Parameters

1. `Address`-查询的地址
2. `string`-查询的起始key
3. `uint`-查询数量

#### Example Parameters

```
"params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf", "key", 1
 ]
```

#### Returns

`Array`-获取的value值列表

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_queryAccountData",
    "params": [
        "0xb5f5758ba45ca7db85ad405d241641da867384a3eefeb80973d3ddc51e176acf", "key", 1
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [10] 
}
```

### Gch_checkPointAt [⇧]()

查询块高的checkpoint

#### Parameters

1. `uint`-查询的块高

#### Example Parameters

```
"params": [
        1
 ]
```

#### Returns

`Object`- checkpoint的块信息

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_checkPointAt",
    "params": [
         1
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "Hash": "0xa8752696d2c62f1915839847d1b4cabc47db12bc6d53a23b3737cc3ddec99cf9",
        "Height": 514159,
        "PreHash": "0xdee31c46af9e65105b1390cc8a8a6bd61a4623a465db3de52ce71c877deb905a",
        "Elapsed": 2800,
        "ProveValue": "A//oOoZq6zTtlFVUcQwk4e1vYSMYCZasThIs/842T8qPixAI7HeelZ/L5f2YxgYqoQEgB5BKwxzOL0Ti1FrJ7dH1X7TtCyTOWeRBms/f7ygY",
        "TotalQN": 2469205,
        "CurTime": 1582620197058,
        "Castor": "p8aoUAYVO3v3DCJfuyQQ1HZazfIKiKnRi/G7+0w+w1Y=",
        "Group": "0x9e66b6859fdd1b6830a4f470293c7e6d4983c73436ee4db88f5ca5e0cdbc30a1",
        "Signature": "C+vl1kgE2wU9VIYSqgB2wwGnVS86Guw92dbARBLIEiEB",
        "Nonce": 1,
        "TxTree": "0x42be24c072dbeeb9aed6fe5bacd64932c9c9075fa2baac37e1e2f992f6bce808",
        "ReceiptTree": "0xaf8664df32859cf3df3f1417c96ec77c046683e44111f5faaf8619342ed20af6",
        "StateTree": "0x01e6f10b0cd887258ff8ff4dfc50d32dc3fd4579342113f5d26778102119c3d0",
        "ExtraData": null,
        "Random": "ErDOneqsFnKgxD8x2QyR5MLNYRBOOb3Bp3F6RyjjDyUB",
        "GasFee": 0
    }
}
```

### Gch_latestCheckPoint [⇧]()

查询最新的checkpoint

#### Parameters

#### Example Parameters

```
"params": [
        
 ]
```

#### Returns

`Object`-checkpoint的块信息

#### Example

```
# Request
import json
import requests
url = 'http://39.101.192.249:8101/'
payload = {
    "method": "Gch_latestCheckPoint",
    "params": [
         
    ],
    "jsonrpc": "2.0",
    "id": 1
}
headers = {
  'Content-Type': 'application/json'
}
response = requests.post(url, headers=headers, data=json.dumps(payload))
print(response.json())
// Response
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "Hash": "0xa8752696d2c62f1915839847d1b4cabc47db12bc6d53a23b3737cc3ddec99cf9",
        "Height": 514159,
        "PreHash": "0xdee31c46af9e65105b1390cc8a8a6bd61a4623a465db3de52ce71c877deb905a",
        "Elapsed": 2800,
        "ProveValue": "A//oOoZq6zTtlFVUcQwk4e1vYSMYCZasThIs/842T8qPixAI7HeelZ/L5f2YxgYqoQEgB5BKwxzOL0Ti1FrJ7dH1X7TtCyTOWeRBms/f7ygY",
        "TotalQN": 2469205,
        "CurTime": 1582620197058,
        "Castor": "p8aoUAYVO3v3DCJfuyQQ1HZazfIKiKnRi/G7+0w+w1Y=",
        "Group": "0x9e66b6859fdd1b6830a4f470293c7e6d4983c73436ee4db88f5ca5e0cdbc30a1",
        "Signature": "C+vl1kgE2wU9VIYSqgB2wwGnVS86Guw92dbARBLIEiEB",
        "Nonce": 1,
        "TxTree": "0x42be24c072dbeeb9aed6fe5bacd64932c9c9075fa2baac37e1e2f992f6bce808",
        "ReceiptTree": "0xaf8664df32859cf3df3f1417c96ec77c046683e44111f5faaf8619342ed20af6",
        "StateTree": "0x01e6f10b0cd887258ff8ff4dfc50d32dc3fd4579342113f5d26778102119c3d0",
        "ExtraData": null,
        "Random": "ErDOneqsFnKgxD8x2QyR5MLNYRBOOb3Bp3F6RyjjDyUB",
        "GasFee": 0
    }
}
```

[ Add a custom footer](https://github.com/0xchain/0xchain/wiki/_new?wiki[name]=_Footer)