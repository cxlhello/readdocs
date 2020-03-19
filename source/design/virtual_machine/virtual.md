# 虚拟机和智能合约

## 智能合约概述
账户
  - 两类账户（它们共用同一个地址空间）： 外部账户 由公钥-私钥对（也就是人）控制； 合约账户 由和账户一起存储的代码控制. 外部账户的地址是由公钥决定的，而合约账户的地址是在创建该合约时确定的（这个地址通过合约创建者的地址和从该地址发出过的交易数量计算得到的，也就是所谓的“nonce”） 无论帐户是否存储代码，这两类账户对 TVM 来说是一样的。 每个账户都有一个键值对形式的持久化存储。其中 key 和 value 的长度都是256位，我们称之为存储。

交易
  - 交易可以看作是从一个帐户发送到另一个帐户的消息。它能包含数据和ZV币。
  - 如果目标账户含有代码，此代码会被执行，并以 payload 作为入参。
  - 如果目标账户是零账户，此交易将创建一个新合约 。
  - 合约地址是通过合约创建者的地址和从该地址发出过的交易数量(nonce)计算得到的。这个用来创建合约的交易的 payload 会被转换为 TVM 字节码并执行。执行的输出将作为合约代码被永久存储。这意味着，为创建一个合约，你不需要发送实际的合约代码，而是发送能够产生合约代码的代码。

智能合约的生命周期

 ![](recycle.png)
 
一个简单的智能合约：
```
class Storage(object):  
    def __init__(self):  
        self.data = 0  
    @register.public(int)  
    def set_data(self, num):  
        self.data = num  
```
合约主要由合约代码和合约数据两部分组成，它们都位于部署合约时生成的地址上。示例合约定义了一个名为Storage的合约，它提供了self.data这个数据的状态存储，并提供了一个set_data接口来修改它。该合约只能提供简单的功能：它能允许任何人在合约中存储一个单独的数字，并且这个数字可以被世界上任何人访问。同时，任何人都可以再次调用 set_data，传入不同的值，覆盖你的数字，但是这个数字仍会被存储在区块链的历史记录中。

## 虚拟机概述

链内置的虚拟机是轻量级的Python解释器，并且针对智能合约使用场景，做了针对性优化、裁剪、改动。

虚拟机执行合约需要消耗资源，防止被用户恶意调用合约，使用Gas机制控制资源消耗。

与CPython的异同点：
主要删减的功能：
  - 禁止使用float类型
  - 禁止使用IO相关功能。io module
  - 禁止使用线程功能。threading module
  - 禁止使用eval、exec功能。
  - 禁止使用网络相关功能。sockect module
  - 禁止使用os module。

主要添加的功能：
  - 内置变量this：表示当前合约的地址(str)。
  - 内置变量msg：合约运行时的相关信息，msg.sender表示调用者地址，msg.value表示调用时带入的资金（联盟链中该值永远为0）。
  - 内置变量register：方法注册器，注册后的方法才可被外部调用。
  - 内置类型zdict：为存储适配的存储键值对的内置类型。
  - 内置类型Event：该类型实例的emit方法可打印数据到交易回执。
  
 ## 合约说明
 
###智能合约与链交互的其他方法
```
1.import account  
2.import block  
3.  
4.''''' 
5.@str: 要查询的地址 
6.returns(int) 返回地址的余额 
7.'''  
8.account.get_balance(str)  
9.  
10.''''' 
11.@str: 要转账的地址 
12.@int: 要转账的金额 
13.returns(bool) 返回转账是否成功 
14.'''  
15.account.transfer(str, int)  
16.  
17.''''' 
18.@int: 要查询的块高 
19.returns(str) 返回查询的块hash 
20.'''  
21.block.blockhash(int)  
22.  
23.''''' 
24.returns(str) 返回当前块高 
25.'''  
26.block.number()

```
 
### 合约支持的数据存储
```
1.class Foo(object):  
2.    def __init__(self):# 基础类型存储  
3.        self.a = {"1":2,"2":3} # Error  
4.        self.b = (1,2,3) # Error  
5.        self.c = [1,2,3,4] # Error  
6.        self.d = 1  
7.        self.e = "abc"  
8.        self.f = True  
9.        # map类型存储  
10.        self.g = zdict()  
11.        self.g[1] = "abc" # Error  
12.        self.g["1"] = "abc"  
13.        self.g["key1"] = "abc"  
14.        self.g["key2"] = True 
```

## 智能合约示例
### 积分合约：
```
1.TransferEvent = Event("transfer")  
2.class Token(object):  
3.    def __init__(self):  
4.        self.name = "TTT"  
5.        self.symbol = "TTT"  
6.        self.decimal = 2  
7.        self.totalSupply = 1000000  
8.        self.balanceOf = zdict()  
9.        self.allowance = zdict()  
10.        self.balanceOf[msg.sender] = self.totalSupply  
11.  
12.    def _transfer(self, _from, _to, _value):  
13.        if _to not in self.balanceOf:  
14.            self.balanceOf[_to] = 0  
15.        if _from not in self.balanceOf:  
16.            self.balanceOf[_from] = 0  
17.        # Whether the account balance meets the transfer amount  
18.        if self.balanceOf[_from] < _value:  
19.            return False  
20.        # Check if the transfer amount is legal  
21.        if _value <= 0:  
22.            return False  
23.        # Transfer  
24.        self.balanceOf[_from] -= _value  
25.        self.balanceOf[_to] += _value  
26.        return True  
27. 
28.    @register.public(str, int)  
29.    def transfer(self, _to, _value):  
30.        if self._transfer(msg.sender, _to, _value):  
31.            TransferEvent.emit(msg.sender, _to, _value)  
32.        else:  
33.            raise Exception("")  
34. 
35.    @register.public(str, int)  
36.    def approve(self, _spender, _value):  
37.        if _value <= 0:  
38.            raise Exception('')  
39.        if msg.sender not in self.allowance:  
40.            self.allowance[msg.sender] = zdict()  
41.        self.allowance[msg.sender][_spender] = _value  
42. 
43.    @register.public(str, str, int)  
44.    def transfer_from(self, _from, _to, _value):  
45.        if _value > self.allowance[_from][msg.sender]:  
46.            raise Exception('')  
47.        self.allowance[_from][msg.sender] -= _value  
48.        if self._transfer(_from, _to, _value):  
49.            TransferEvent.emit(_from, _to, _value)  
50.        else:  
51.            raise Exception("")  
52. 
53.    @register.public(int)  
54.    def burn(self, _value):  
55.        if _value <= 0:  
56.            raise Exception('')  
57.        if self.balanceOf[msg.sender] < _value:  
58.            raise Exception('')  
59.        self.balanceOf[msg.sender] -= _value  
60.        self.totalSupply -= _value
```

### 猜拳合约
```
1.import block  
2.import account  
3.  
4.  
5.class ContractFingerGuessing():  
6.    def __init__(self):  
7.        self.games = zdict()  
8. 
9.    @register.public(str, str, str)  
10.    def make_gestures(self, session, account, gestures):  
11.        if not self.games.has_key(session):  
12.            self.games[session] = {}  
13.        if len(self.games[session].items()) < 2:  
14.            self.games[session].account.gestures = gestures  
15. 
16.    @register.public(str)  
17.    def game_result(self, session):  
18.        if self.games.has_key(session):  
19.            gestures_list = self.games[session].items()  
20.            if len(gestures_list) == 2:  
21.                a = gestures_list[0][1].gestures  
22.                b = gestures_list[1][1].gestures  
23.                if (a == "rock" and b == "scissors") or (a == "scissors" and b == "paper") | | (  
24.                        a == "paper" and b == "rock"):  
25.                    self.games[session].winner = gestures_list[0][0]  
26.                else:  
27.                    self.games[session].winner = gestures_list[1][0]
```


### 众筹合约：
```

1.import account  
2.import block  
3.  
4.  
5.class CrowdFunding():  
6.    def deploy(self):  
7.        self.funding_goal = 10000  
8.        self.funding = 0  
9.        self.max_block_number = block.number() + 1000  
10.        self.vote_dict = zdict  
11.        self.on_sale = True  
12.        self.owner = msg.sender  
13. 
14.    @register.public()  
15.    def sale(self):  
16.        if self.max_block_number < block.number():  
17.            self.on_sale = False  
18.        if not self.on_sale:  
19.            raise Exception("not on sale")  
20.        value = msg.value  
21.        sender = msg.sender  
22.        self.funding = self.funding + value  
23.        if self.funding > self.funding_goal:  
24.            value = self.funding_goal - self.funding  
25.            self.on_sale = False  
26.            account.transfer(sender, value)  
27.            return  
28.        balance = self.vote_dict.get(sender, 0)  
29.        self.vote_dict[sender] = balance + value  
30.        print(self.vote_dict)  
31. 
32.    @register.public(str)  
33.    def withdraw(self, addr):  
34.        if self.owner != msg.sender:  
35.            return  
36.        if not self.on_sale and self.funding >= self.funding_goal:  
37.            account.transfer(addr, self.funding)  
38. 
39.    @register.public()  
40.    def failed(self):  
41.        if not self.on_sale and self.funding < self.funding_goal and self.funding >= account.get_balance(this):  
42.            for k, v in self.vote_dict.items():  
43.                account.transfer(k, v)
```

 
 