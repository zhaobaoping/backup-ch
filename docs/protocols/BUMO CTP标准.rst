BUMO CTP标准
==============

概述
----

CTP1.0（Contract Token Protocol）指基于 BUMO 合约发行 token 的协议标准。本协议提供了转移 token 的接口，第三方软件可以通过接口使用 token。
BUMO 智能合约是用 javascript 实现的，包含初始化函数 ``init`` 和两个入口函数 ``main``、``query``。``init`` 函数主要负责合约创建时的初始化，``main`` 函数主要负责数据写入，``query`` 函数主要负责数据查询。

目标
--------

通过CTP协议让其他应用程序方便地调用接口，在 BUMO 上使用任何 token。其他应用程序包括钱包和交易所等。

Token属性参数
-------------

Token 属性可以通过合约的 ``tokenInfo`` 功能函数查询到，存储在智能合约的账号里。Token 属性包含以下内容：

+--------------+------------------------------------+
| 变量         | 描述                               |
+==============+====================================+
| ctp          | Contract Token Protocol 版本       |
+--------------+------------------------------------+
| name         | token 名称                         |
+--------------+------------------------------------+
| symbol       | token 符号                         |
+--------------+------------------------------------+
| decimals     | token 小数位数                     |
+--------------+------------------------------------+
|totalSupply   | token 总量                         |
+--------------+------------------------------------+
|contractOwner | token 所有者                       |	
+--------------+------------------------------------+	 


.. note:: 

 - name：推荐使用单词全拼，每个首字母大写，如 Demo Token。
 - symbol：推荐使用大写首字母缩写，如 DT。
 - decimals：小数位在 0~8 的范围，0 表示无小数位。
 - totalSupply：范围是 1~2^63-1。


函数
-----

BUMO CTP标准中的函数包括 `contractInfo`_、`name`_、`symbol`_、`decimals`_、`totalSupply`_、`balanceOf`_、`transfer`_、`transferFrom`_、`approve`_、`assign`_、`changeOwner`_、`allowance`_。

contractInfo
^^^^^^^^^^^^^

``contractInfo`` 函数用于返回 token 的基本信息，其入口函数为 ``query``。

**参数json结构** 

::
 
 {
    "method":"contractInfo"
 }

**函数**

::
 
 function contractInfo()

**返回值**

::

 {
    "result":{
        "type": "string",
        "value": {
            "contractInfo": {
                "ctp": "1.0",
                "name": "cccpt-bu",
                "symbol": "CBG",
                "decimals": 0,
                "totalSupply": "100000",
                "contractOwner": "buQBv4pqtNMs6ueBhx7mJULhAFYV3rSHo2Zg",
                "balance": "100000"
            }
        }
    }
 } 

name
^^^^^

``name`` 函数用于返回 token 的名称，其入口函数是 ``query``。

**参数json结构** 

::
 
 {
    "method":"name"
 }

**函数**

::
 
 function name()

**返回值**

::

 {
    "result":{
        "name":"XXXCOIN"
    }
 } 

symbol
^^^^^^^

``symbol`` 函数用于返回 token 的符号，其入口函数是 ``query``。

**参数json结构** 

::
 
 {
    "method":"symbol"
 }

**函数**

::
 
 function symbol()

**返回值**

::

 {
    "result":{
        "symbol":"XXX"
    }
 } 

decimals
^^^^^^^^^

``decimals`` 函数用于返回 token 使用的小数点后几位，比如 5 表示分配 token 数量为 100000，其入口函数为 ``query``。

**参数json结构** 

::
 
 {
    "method":"decimals"
 }

**函数**

::
 
 function decimals()

**返回值**

::

 {
    "result":{
        "decimals":5
    }
 } 


totalSupply
^^^^^^^^^^^^^

``totalSupply`` 函数用于返回 token 的总供应量，其入口函数为 ``query``。

**参数json结构** 

::
 
 {
    "method":"totalSupply"
 }

**函数**

::

 function totalSupply()

**返回值**

::

 {
    "result":{
        "totalSupply":"10000000000000000000"
    }
 } 

balanceOf
^^^^^^^^^^

``balanceOf`` 函数用于返回 owner 账户的账户余额，其入口函数为 ``query``。

**参数json结构** 

::
 
 {
      "method":"balanceOf",
      "params":{
        "address":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj"
    }
 }

**参数说明**

address: 账户地址。

**函数**

::
 
 function balanceOf()

**返回值**

::

 {
    "result":{
        "balanceOf":"100000000000000"
    }
 } 

transfer
^^^^^^^^

``transfer`` 函数用于转移数量为 value 的 token 到目的地址 to，并且必须触发 log 事件。 如果资金转出账户没有足够的 token 来支出，该函数应该被 throw，其入口函数为 ``main``。

**参数json结构** 

::
 
 {
    "method":"transfer",
    "params":{
        "to":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj",
        "value":"1000000"
 }

**参数说明**

to: 目标账户地址。

value: 转移数量（字符串类型）。

**函数**

::
 
 function transfer(to, value)

**返回值**

true或者抛异常。

transferFrom
^^^^^^^^^^^^^

``transferFrom`` 函数用于从地址 from 发送数量为 value 的 token 到地址 to，必须触发 log 事件。 在 transferFrom 之前，from 必须已经调用过 approve 向 to 授权了额度。
如果 from 账户没有足够的 token 来支出或者 from 授权给 to 的额度不足，该函数应该被 throw，其入口函数为 ``main``。

**参数json结构** 

::
 
 {
    "method":"transferFrom",
    "params":{
        "from":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj",
        "to":"buQYH2VeL87svMuj2TdhgmoH9wSmcqrfBner",
        "value":"1000000"
    }
 }

**参数说明**

from: 源账户地址。

to: 目标账户地址。

value: 转移数量（字符串类型）。

**函数**

::
 
 function transferFrom(from,to,value)

**返回值**

true或者抛异常。

approve
^^^^^^^^

``approve`` 函数用于授权账户 spender 从交易发送者账户转出数量为 value 的 token，其入口函数为 ``main``。

**参数json结构** 

::
 
 {
    "method":"approve",
    "params":{
        "spender":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj",
        "value":"1000000"
    }
 }

**参数说明**

spender: 账户地址。

value: 被授权可转移数量（字符串类型）。

**函数**

::
 
 function approve(spender, value)

**返回值**

true或者抛异常。

assign
^^^^^^^

``assign`` 函数用于实现合约 token 拥有者向 to 分配数量为 value 的 token，其入口函数为 ``main``。

**参数json结构** 

::
 
 {
    "method":"assign",
    "params":{
        "to":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj",
        "value":"1000000"
    }
 }

**参数说明**

to: 收账账户地址。

value: 分配数量（字符串类型）。

**函数**

::
 
 function assign(to, value)

**返回值**

true或者抛异常。

changeOwner
^^^^^^^^^^^^

``changeOwner`` 函数用于将合约 token 的拥有权（默认拥有者为合约资产的创建账户）转移给 address，只有合约 token 拥有者才能执行此权限，其入口函数为 ``main``。

**参数json结构** 

::
 
 {
    "method":"changeOwner",
    "params":{
        "address":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj"
    }
 }

**参数说明**

address: 账户地址。

**函数**

::
 
 function changeOwner(address)

**返回值**

true或者抛异常。

allowance
^^^^^^^^^^

``allowance`` 函数用于返回 spender 仍然被允许从 owner 提取的金额，其入口函数为 ``query``。

**参数json结构** 

::
 
 {
    "method":"allowance",
    "params":{
        "owner":"buQnTmK9iBFHyG2oLce7vcejPQ1g5xLVycsj",
        "spender":"buQYH2VeL87svMuj2TdhgmoH9wSmcqrfBner"
    }
 }

**参数说明**

owner: 拥有者的账户地址。

spender: 花费者的账户地址。

**函数**

::
 
 function allowance(owner, spender)

**返回值**

::
 
 {
    "result":{
        "allowance":"1000000",
    }
 } 

入口函数
---------

BUMO 智能合约提供了 `初始化函数 init`_、 `入口函数 main`_ 和 `入口函数 query`_。

初始化函数 init
^^^^^^^^^^^^^^^^^^^

``init`` 函数主要负责合约创建时的初始化，下面是该函数的函数说明、参数结构、参数说明和返回值。

**函数**

::

 function init(input_str){
 }

**参数json结构**

::

 {
    "params":{
        "name":"RMB",
        "symbol":"CNY",
        "decimals":8,
        "totalSupply":"5000000000000"
    }
 }

**参数说明**

name: 资产名称。

symbol: 资产符号。

decimals：小数位数。

totalSupply：字符串格式，发行 token 总数。例如，发行 50000 个 token，其 totalSupply 值为 50000 * 100000000。

**返回值**

true或者抛异常。

入口函数 main
^^^^^^^^^^^^^^^^^

``main`` 函数主要负责数据写入，其中包含了 ``transfer``、``transferFrom``、``approve``、``assign``、
``changeOwner`` 等接口，下面是 ``main`` 的函数体。

::

 function main(input_str){
    let input = JSON.parse(input_str);

    if(input.method === 'transfer'){
        transfer(input.params.to, input.params.value);
    }
    else if(input.method === 'transferFrom'){
        transferFrom(input.params.from, input.params.to, input.params.value);
    }
    else if(input.method === 'approve'){
        approve(input.params.spender, input.params.value);
    }
    else if(input.method === 'assign'){
        assign(input.params.to, input.params.value);
    }
    else if(input.method === 'changeOwner'){
        changeOwner(input.params.address);
    }
    else{
        throw '<undidentified operation type>';
    }
 }

入口函数 query
^^^^^^^^^^^^^^^^^^

``query`` 函数主要负责数据查询，其中包含了 ``name``、``symbol``、``decimals``、``totalSupply``、
``contractInfo``、``balanceOf``、``allowance`` 等接口，下面是 ``query`` 的函数体。

::

 function query(input_str){
    loadGlobalAttribute();

    let result = {};
    let input  = JSON.parse(input_str);
    if(input.method === 'name'){
        result.name = name();
    }
    else if(input.method === 'symbol'){
        result.symbol = symbol();
    }
    else if(input.method === 'decimals'){
        result.decimals = decimals();
    }
    else if(input.method === 'totalSupply'){
        result.totalSupply = totalSupply();
    }
    else if(input.method === 'contractInfo'){
        result.contractInfo = contractInfo();
    }
    else if(input.method === 'balanceOf'){
        result.balance = balanceOf(input.params.address);
    }
    else if(input.method === 'allowance'){
        result.allowance = allowance(input.params.owner, input.params.spender);
    }
    else{
       	throw '<unidentified operation type>';
    }

    log(result);
    return JSON.stringify(result);
 }
