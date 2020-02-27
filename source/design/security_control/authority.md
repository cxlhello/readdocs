# Chiron 权限文档

## 1.chiron 权限系统简介

chiron 权限系统是利用智能合约来控制的，在本权限系统中，设计上共采用`7`个合约，他们分别是：

**`PermissionsUpgradable合约`**：只能通过guardian用户来调用的合约，一般在初始化的时候来绑定各个合约地址或者当`PermissionImplementation`合约逻辑更改来升级合约时候使用。

**`PermissionsInterface合约`**：接口合约，用户对权限的操作需调用该合约中封装的接口来调用，该合约会调用`PermissionsImplementation`合约来进行进一步调用。

**`PermissionsImplementation合约`**：通过上面的`PermissionsInterface`合约来进行统一调用的，该合约进而对底层的合约进行调用，考虑到以后存在由于逻辑变动而要升级合约的情况，该合约是可以被`guardian`重新绑定以进行升级操作。

**`OrgMgr合约`**：对组织进行存储的合约，包含组织列表，组织状态等等。`PermissionsImplementation合约`对组织的操作最终都会回到该合约中操作。通过`PermissionsImplementation合约`调用。

**`VoteMgr合约`**：进行投票管理的合约，该合约中存储投票者和各投票事项。一般在联盟层面上的操作都需要联盟管理员进行申请和投票两步操作才可以。通过`PermissionsImplementation合约`调用。

**`AccountMgr合约`**：对账户进行管理的合约。通过`PermissionsImplementation合约`调用。

**`NodeMgr合约`**：对节点进行管理的合约，可以指定某个节点是否有挖矿权限。通过`PermissionsImplementation合约`调用。


## 2.chiron permission APIs
**↓↓↓↓↓以下基于联盟内的操作都需要通过联盟管理员操作，且需要两步完成，即申请+批准↓↓↓↓↓**
#### add_org
用来在联盟中添加一个新组织时候使用。该方法仅可以被联盟管理员调用
##### Parameters
`_org_id`:新添加的组织id
`_account`:新添加组织中的账户id，该账户会被自动指定为该组织的管理员
`_node_id`:新添加组织中的节点id

* * *
#### approve_org
用来在联盟中投票通过一个新组织时候使用。该方法仅可以被联盟管理员调用，与前面的`add_org`方法是相关联的操作，前面通过`add_org`添加的组织需要联盟管理员调用此方法进行投票投票且超半数方可成功加入。
##### Parameters
`_org_id`:需要approve操作的组织id
`_account`:需要approve操作组织中的账户id，该账户会被自动指定为该组织的管理员
`_node_id`:需要approve操作组织中的节点id

* * *
#### update_org_status
由联盟管理员执行，当需要更改非联盟管理员组织的状态的时候使用。
##### Parameters
`_org_id`:需要更改状态的组织id
`_action`:操作符，`1`表示暂停组织，`2`表示将组织从已暂停的状态中恢复为激活状态

* * *
#### approve_org_status
由联盟管理员帐户执行，并用于批准组织状态更改建议。从网络管理员那里获得多数批准后，组织状态就会更新。
##### Parameters
`_org_id`:需要更改状态的组织id
`_action`:操作符，`1`表示暂停组织，`2`表示将组织从已暂停的状态中恢复为激活状态

* * *
#### assign_alliance_admin
由联盟管理员帐户执行，用于向联盟组织中添加一个新的联盟管理员账户时使用
##### Parameters
`_org_id`:联盟管理员组织id
`_account`:需要指派为联盟管理员的账户id

* * *
#### approve_alliance_admin
由联盟管理员帐户执行，用于批准向联盟管理员组织中新加管理员的建议。获得多数联盟管理员批准后，组织中便会增加一个有效的联盟管理员账户。
##### Parameters
`_org_id`:联盟管理员组织id
`_account`:需要批准为联盟管理员的账户id

* * *
#### add_miner_node
由联盟管理员帐户执行，用于向联盟中新添加一个挖矿节点时使用
##### Parameters
`_node_id`:该节点的id
`_org_id`:需要添加到目的组织的id
`_miner_role`:表示矿工类型。`1`表示提案矿工，`2`表示验证矿工
`_vrf_pk`:vrf_pk
`_bls_pk`:bls_pk
`_weight`:表示矿工权重

* * *
#### approve_miner_node
由联盟管理员帐户执行，用于批准向联盟中新添加一个挖矿节点时使用，同样需要得到多数的联盟管理员批准后方可生效。
##### Parameters
`_node_id`:要批准的该节点的id
`_org_id`:要批准的需要添加到目的组织的id
`_miner_role`:要批准的矿工类型。`1`表示提案矿工，`2`表示验证矿工
`_vrf_pk`:要批准的vrf_pk
`_bls_pk`:要批准的bls_pk
`_weight`:要批准的矿工权重

* * *
#### assign_node_to_miner
由联盟管理员帐户执行，用于将联盟中某个已存在的普通同步节点指定为矿工节点
##### Parameters
`_node_id`:该节点的id
`_org_id`:该节点所存在的组织的id
`_miner_role`:要分配的矿工类型。`1`表示提案矿工，`2`表示验证矿工
`_vrf_pk`:vrf_pk
`_bls_pk`:bls_pk
`_weight`:矿工权重

* * *
#### approve_node_to_miner
由联盟管理员帐户执行，用于批准将联盟中某个已存在的普通同步节点指定为矿工节点的请求，需要得到多数的联盟管理员批准后方可生效。
##### Parameters
`_node_id`:要批准的该节点的id
`_org_id`:要批准的需要添加到目的组织的id
`_miner_role`:要批准的矿工类型。`1`表示提案矿工，`2`表示验证矿工
`_vrf_pk`:要批准的vrf_pk
`_bls_pk`:要批准的bls_pk
`_weight`:要批准的矿工权重

* * *
#### update_miner_status
由联盟管理员帐户执行，用于更改节点的矿工状态时使用
##### Parameters
`_node_id`:要批准的该节点的id
`_org_id`:要批准的需要添加到目的组织的id
`_action`:操作符，`1`表示暂停矿工，`2`表示矿工从已暂停的状态中恢复为激活状态

* * *
#### approve_miner_status
由联盟管理员帐户执行，用于批准更改节点的矿工状态请求时使用。当得到多数的联盟管理员批准后方可生效。
##### Parameters
`_node_id`:要批准的该节点的id
`_org_id`:要批准的需要添加到目的组织的id
`_action`:操作符，`1`表示暂停矿工，`2`表示将矿工从已暂停的状态中恢复为激活状态

* * *

**↓↓↓↓↓以下基于组织内的操作都需要通过组织管理员操作↓↓↓↓↓**

#### add_account
由组织管理员帐户执行，用于向组织中新添加一个账户时使用。
##### Parameters
`_account`:新添加的账户id
`_org_id`:新添加的账户所从属的组织，该组织必须存在且为`approved`状态
`_access`:读写权限，分为四个等级。用户可指定`0`,`1`,`2`三个等级，高级别兼具有低级别的权限：
`0`:ACCESS_READONLY，只读权限
`1`:ACCESS_TRANSACT，可以发送交易  
`2`:ACCESS_CONTRACT_DEPLOY，可以部署合约  
`3`:ACCESS_FULL_ACCESS，全访问权限
`_is_admin`:bool值，表示该账户是否为管理员
**注意：如果`_is_admin`设置为true时，`_access`的值不能为`0`只读状态**

* * *
#### update_account_status
由组织管理员帐户执行，用于更改组织中某个账户的状态时使用。
##### Parameters
`_account`:要更改的账户id
`_org_id`:要更改的账户所从属的组织
`_action`:操作符，`1`表示暂停使用该账户，`2`表示将已暂停的账户恢复为激活状态

* * *
#### update_account_access
由组织管理员帐户执行，用于更改组织中某个账户的读写权限时使用。
##### Parameters
`_account`:要更改的账户id
`_org_id`:要更改的账户所从属的组织
`_access`:想要更改的目的读写权限，分为四个等级。用户可指定`0`,`1`,`2`三个等级，高级别兼具有低级别的权限：
`0`:ACCESS_READONLY，只读权限
`1`:ACCESS_TRANSACT，可以发送交易  
`2`:ACCESS_CONTRACT_DEPLOY，可以部署合约
`3`:ACCESS_FULL_ACCESS，全访问权限

* * *
#### add_node
由组织管理员帐户执行，用于向组织中新增一个同步节点时使用。
##### Parameters
`_node_id`:新增的节点id
`_org_id`:新增节点所从属的组织id

* * *
#### update_node_status
由组织管理员帐户执行，用于更改组织中某个节点的状态时使用
##### Parameters
`_node_id`:新增的节点id
`_org_id`:新增节点所从属的组织id
`_action`:操作符，`1`表示暂停该节点，`2`表示将已暂停的节点恢复激活状态

账户权限说明：

| 权限 |值  |说明|
| --- | --- | --- |
| ACCESS READONLY | 0 | 只读权限 |
| ACCESS TRANSACT | 1 | 发送交易权限，同时具备0权限 |
| ACCESS CONTRACT DEPLOY | 2 | 部署合约权限，同时具备0，1权限 |
| ACCESS FULL ACCESS | 3 | 全访问权限，为联盟管理员特有，同时具备0，1，2权限 |

节点矿工类型说明：

| 矿工类型 | 值 |
| --- | --- |
| PROPOSAL MINER | 1 |
| VERIFY MINER | 2 |

节点状态说明：

| 状态描述 | 值 |
| --- | --- |
| NOT IN LIST | 0 |
| PENDING APPROVAL | 1 |
| ACTIVE | 2 |
| SUSPENDED | 3 |

矿工状态说明：

| 状态描述 | 值 |
| --- | --- |
| MINER NO ACTION | 0 |
| MINER PENDING ACTIVATE | 1 |
| MINER ACTIVATED | 2 |
| MINER PENDING ABORT | 3 |
| MINER ABORTED | 4 |
| MINER ABORTED REVOKE | 5 |

组织状态说明：

| 状态描述 | 值 |
| --- | --- |
| NOT IN LIST | 0 |
| PROPOSED | 1 |
| APPROVED | 2 |
| PENDING SUSPENSION | 3 |
| SUSPENDED | 4 |
| PENDING SUSPENSION REVOKE | 5 |

账户状态说明：

| 状态描述 | 值 |
| --- | --- |
| NOT IN LIST | 0 |
| PENDING APPROVAL | 1 |
| ACTIVE | 2 |
| SUSPENDED | 3 |

投票类型：

| 投票类型 | 值 |
| --- | --- |
| VOTE OP ADD ACTIVITY ORG | 1 |
| VOTE OP SUSPEND ORG | 2 |
| VOTE OP REVOKE SUSPEND ORG | 3 |
| VOTE OP ASSIGN ALLIANCE ADMIN | 4 |
| VOTE OP REMOVE ALLIANCE ADMIN | 5 |
| VOTE OP ADD MINER NODE | 6 |
| VOTE OP ASSIGN NODE TO MINER | 7 |
| VOTE OP UPDATE MINER STATUS | 8 |