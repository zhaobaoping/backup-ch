BUMO 智能合约开发指南
===============

本文档主要描述智能合约开发相关内容，包括 `合约定义`_、智能合约 `语法说明`_、系统提供的 `内置函数`_、`内置变量`_、`异常处理`_、`示例`_ 等。

合约定义
--------

合约是一段 ``JavaScript`` 代码。
合约的初始化函数是 ``init``，执行的入口函数是 ``main`` 和 ``query``，其中 ``init`` 和 ``main`` 的定义是必需的。
以上函数的入口参数 **input** 为字符串型，调用合约的时候需要指定。


.. code:: javascript
 
   "use strict"; 
   function init(input) 
   { 
     /* init whatever you want */ return;
   }

   function main(input) 
   { 
    /* do whatever you want */ return; 
   }

   function query(input) 
   { 
     /* query whatever you want, but should return what you query */ return input; 
   }


语法说明
--------- 



Bumo 智能合约使用 ``JaveScript`` 语言进行编写，为了方便开发者更规范、更安全地开发合约，在进行合约语法检测时使用了 JSLint 进行限制。
详情请参考： `ContractRules.md <https://github.com/bumoproject/bumo/blob/master/src/web/jslint/ContractRules.md>`_ 。

语法规范
^^^^^^^^^

智能合约中的语法规范包括以下内容：

- 严格检测声明，所有的源码在开始必须要添加 **"use strict";** 字段。
- 语句块内尽量使用 **let** 声明变量。
- 使用 **===** 代替 **==** 判断比较；使用 **!==** 代替 **!=** 比较。
- 语句必须以 **;** 结束。
- 语句块必须用 **{}** 括起来，且禁止空语句块。
- **for** 的循环变量和初始变量需在条件语句块之前声明，每次使用时重新赋值。
- 禁止使用 **++** 和 **--** ，使用 **+=** 和 **-=** 替代。
- 禁止使用 **eval**、**void**、**this** 关键字。
- 禁止使用 **new** 创建 **Number**、**String**、**Boolean** 对象，可以使用其构造调用来获取对象。
- 禁止使用以下数组关键字创建数组：

::

 "Array", "ArrayBuffer", "Float32Array", "Float64Array", "Int8Array", "Int16Array", "Int32Array", "Uint8Array", "Uint8ClampedArray", "Uint16Array", "Uint32Array"
 let color = new Array(100); //编译报错 
 
 
 //可以使用以下语句替代 new Array(100) 语句; 
 let color = ["red","black"]; 
 let arr = [1,2,3,4];


- 禁止使用 **try**、**catch** 关键字，可以使用 **throw** 手动抛出异常。
- 禁止使用以下关键字：

::

 "DataView", "decodeURI", "decodeURIComponent", "encodeURI", "encodeURIComponent", "Generator","GeneratorFunction", 
 "Intl", "Promise", "Proxy", "Reflect", "System", "URIError", "WeakMap", "WeakSet", "Math", "Date"

检测工具
^^^^^^^^^

检测工具分为本地检测工具和线上检测工具。

使用本地检测工具可下载 `jslint <https://github.com/bumoproject/bumo/tree/master/src/web/jslint>`_，双击目标下的 index.html。

使用线上检测工具可打开 `jslint.html <http://bumo.chinacloudapp.cn:36002/jslint.html>`_。

文本压缩工具
^^^^^^^^^^^^

文本压缩工具分为本地压缩工具和线上压缩工具。

使用本地压缩工具可打开 `jsmin <https://github.com/bumoproject/bumo/tree/master/deploy/jsmin>`_ 。

使用线上压缩工具可打开 `jsmin.html <https://jsmin.51240.com>`_ 。

内置函数
--------

系统提供了一些全局函数，方便用户获取区块链相关信息，或驱动账号发起所有交易（除了设置门限和设置权重这两种类型的操作）。

.. note:: 自定义的函数和变量不要与全局函数和内置变量重名，否则会造成不可控的数据错误。

详情请参考 `合约 <https://github.com/bumoproject/bumo/blob/master/docs/develop_CN.md#合约>`_ 。


函数读写权限
^^^^^^^^^^^^

每个函数都有固定的只读或者可写权限。
只读权限是指不会写数据到区块链，比如获取余额函数 **getBalance** 具有只读权限。
可写权限是指会写数据到区块链，比如转账函数 **payCoin** 具有可写权限。 
在编写智能合约的时候，不同的入口函数拥有不同的调用权限。 ``init`` 和 ``main`` 能调用所有的内置函数。 ``query`` 只能调用具有只读权限的函数，否则在调试或者执行过程中会提示接口未定义。

返回值介绍
^^^^^^^^^^

所有内部函数的调用，如果失败则返回 **false** 或者直接抛出异常终止执行。
如果遇到参数错误，会在错误描述中提示出错的参数位置，这里的位置指参数的索引号，即从 0 开始计数。
例如，parameter 1 表示第 2 个参数错误。如下例子：

::
 
 issueAsset("CNY", 10000); /* 错误描述：Contract execute error,issueAsset parameter 1 should be a string 指第 2 个参数应该为字符串 */

函数详情
^^^^^^^^^

本章节主要介绍智能合约开发过程涉及的一些函数，包括 ``getBalance``、``storageStore``、``storageLoad``、``storageDel``、``getAccountAsset``、
``getBlockHash``、``addressCheck``、``stoI64Check``、``int64Add``、``int64Sub``、
``int64Mul``、``int64Div``、``int64Mod``、``int64Compare``、``toBaseUnit``、``log``、
``tlog``、``issueAsset``、``payAsset``、``payCoin``、``assert``。

getBalance
~~~~~~~~~~~

**函数描述：**

``getBalance`` 函数用于获取账号信息（不包含 metada 和资产信息）。

**函数调用：**

::

 getBalance(address);

**参数说明：**

address：账号地址。

**示例：**


.. code:: javascript

 let balance = getBalance('buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY'); 
 
 /* 权限：只读 
    返回：字符串格式数字 '9999111100000' */

storageStore
~~~~~~~~~~~~

**函数描述：**

``storageStore`` 函数用于存储合约账号的 metadata 信息。

**函数调用：**

::

 storageStore(metadata_key, metadata_value);

**参数说明：**

metadata_key：metadata 的 key 值。

metadata_value：metadata 的 value 值。

**示例：**

.. code:: javascript

 storageStore('abc', 'values'); 
 /* 权限：可写 
    返回：成功返回true, 失败抛异常 */

storageLoad
~~~~~~~~~~~~

**函数描述：**

``storageLoad`` 函数用于获取合约账号的 metadata 信息。

**函数调用：**

::
 
 storageLoad(metadata_key);

**参数说明：**

metadata_key：metadata 的 key 值。

**示例：**


.. code:: javascript
 
 let value = storageLoad('abc'); 
 /* 权限：只读 
    返回：成功返回字符串，如 'values', 失败返回 false 
    本示例得到合约账号中自定数据的 abc 的值*/

storageDel
~~~~~~~~~~~

**函数描述：**

``storageDel`` 函数用于删除合约账号的 metadata 信息。

**函数调用：**

::

 storageDel(metadata_key);

**参数说明：**

metadata_key：metadata 的 key 值。

**示例：**


.. code:: javascript

 storageDel('abc');
 /*
  权限：可写
  返回：成功返回 true, 失败抛异常
  本示例删除本合约账号中自定数据的 abc 的值*/

getAccountAsset
~~~~~~~~~~~~~~~~

**函数描述：**

``getAccountAsset`` 函数用于获取某个账号的资产信息。

**函数调用：**

::

 getAccountAsset(account_address, asset_key);

**参数说明：**

account_address：账号地址。

asset_key：资产属性。

**示例：**


.. code:: javascript


 let asset_key =
 {
 'issuer' : 'buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY',
 'code' : 'CNY'
 };
 let bar = getAccountAsset('buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY', 
 asset_key);
 /*
 权限：只读
 返回：成功返回资产数字如'10000'，失败返回 false
 */


getBlockHash
~~~~~~~~~~~~~

**函数描述：**

``getBlockHash`` 函数用于获取区块信息。

**函数调用：**

::

 getBlockHash(offset_seq);

**参数说明：**

offset_seq：距离最后一个区块的偏移量，最大为1024。

**示例：**


.. code:: javascript

 let ledger = getBlockHash(4);
 /*
 权限：只读
 返回：成功返回字符串，如
 'c2f6892eb934d56076a49f8b01aeb3f635df3d51aaed04ca521da3494451afb3'，
 失败返回 false
 */


addressCheck
~~~~~~~~~~~~~

**函数描述：**

``addressCheck`` 函数用于地址合法性检查。

**函数调用：**

::
 
 addressCheck(address);

**参数说明：**

address：地址参数，类型为字符串型。

**示例：**

.. code:: javascript

 let ret = addressCheck('buQgmhhxLwhdUvcWijzxumUHaNqZtJpWvNsf');
 /*
 权限：只读
 返回：成功返回 true，失败返回 false
 */

stoI64Check
~~~~~~~~~~~~

**函数描述：**

``stoI64Check`` 函数用于字符串数字合法性检查。

**函数调用：**

::

 stoI64Check(strNumber);

**参数说明：**

strNumber：字符串数字参数。

**示例：**

.. code:: javascript

 let ret = stoI64Check('12345678912345');
 /*
 权限：只读
 返回：成功返回 true，失败返回 false
 */

int64Add
~~~~~~~~~~

**函数描述：**

``int64Add`` 函数用于64 位加法运算。

**函数调用：**

::

 int64Add(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Add('12345678912345', 1);
 /*
 权限：只读
 返回：成功返回字符串 '12345678912346', 失败抛异常
 */

int64Sub
~~~~~~~~~

**函数描述：**

``int64Sub`` 函数用于64位减法运算。

**函数调用：**

::

 int64Sub(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Sub('12345678912345', 1);
 /*
 权限：只读
 返回：成功返回字符串 '123456789123464'，失败抛异常
 */

int64Mul
~~~~~~~~~~

**函数描述：**

``int64Mul`` 函数用于64位乘法运算。

**函数调用：**

::

 int64Mul(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Mul('12345678912345', 2);
 /*
 权限：只读
 返回：成功返回字符串 '24691357824690'，失败抛异常
 */

int64Div
~~~~~~~~~~

**函数描述：**

``int64Div`` 函数用于64位除法运算。

**函数调用：**

::

 int64Div(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Div('12345678912345', 2);
 /*
 权限：只读
 返回：成功返回 '6172839456172'，失败抛异常
 */

int64Mod
~~~~~~~~~

**函数描述：**

``int64Mod`` 函数用于64位取模运算。

**函数调用：**

::

 int64Mod(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Mod('12345678912345', 2);
 /*
 权限：只读
 返回：成功返回字符串 '1'，失败抛异常
 */

int64Compare
~~~~~~~~~~~~~

**函数描述：**

``int64Compare`` 函数用于64位比较运算。

**函数调用：**

::

 int64Compare(left_value, right_value);

**参数说明：**

left_value：左值。

right_value：右值。

**示例：**

.. code:: javascript

 let ret = int64Compare('12345678912345', 2);
 /*
 权限：只读
 返回：成功返回数字 1（左值大于右值），失败抛异常
 */

.. note:: 
 
 - 返回值为 1：左值大于右值。
 - 返回值为 0：左值等于右值。
 - 返回值为-1 ：左值小于右值。

toBaseUnit
~~~~~~~~~~~

**函数描述：**

``toBaseUnit`` 函数用于变换单位。

**函数调用：**

::

 toBaseUnit(value);

**参数说明：**

value：被转换的数字，只能传入字符串，可以包含小数点，且小
数点之后最多保留 8 位数字。

**示例：**

.. code:: javascript

 let ret = toBaseUnit('12345678912');
 /*
 权限：只读
 返回：成功会返回乘以 10^8 的字符串，本例返回字符串 '1234567891200000000'，失败抛异常
 */

log
~~~~

**函数描述：**

``log`` 函数用于输出日志。

**函数调用：**

::

 log(info); 

**参数说明：**

info：日志内容。

**示例：**

.. code:: javascript

 let ret = log('buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY');
 /*
 权限：只读
 返回：成功无返回值，失败返回 false
 */

tlog
~~~~~

**函数描述：**

``tlog`` 函数用于输出交易日志，调用该函数会产生一笔交易写在区块上。

**函数调用：**

::

 tlog(topic,args...);

**参数说明：**

topic：日志主题，必须为字符串类型，参数长度为(0,128]。

args...：最多可以包含 5 个参数，参数类型可以是字符串、数值或者布尔类型，每个参数长度为 (0,1024]。


**示例：**

.. code:: javascript

 tlog('transfer',sender +' transfer 1000',true);
 /*
 权限：可写
 返回：成功返回 true，失败抛异常
 */

issueAsset
~~~~~~~~~~~

**函数描述：**

``issueAsset`` 函数用于发行资产。

**函数调用：**

::

 issueAsset(code, amount);

**参数说明：**

code：资产代码。

amount：发行资产数量。


**示例：**

.. code:: javascript

 issueAsset("CNY", "10000");
 /*
 权限：可写
 返回：成功返回 true，失败抛异常 
 */


payAsset
~~~~~~~~~

**函数描述：**

``payAsset`` 函数用于转移资产。

**函数调用：**

::

 payAsset(address, issuer, code, amount[, input]);

**参数说明：**

address：转移资产的目标地址。

issuer：资产发行方。

code：资产代码。

amount：转移资产的数量。

input：可选，合约参数，默认为空字符串。


**示例：**

.. code:: javascript

 payAsset("buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY", 
 "buQgmhhxLwhdUvcWijzxumUHaNqZtJpWvNsf", "CNY", "10000", "{}");
 /*
 权限：可写
 返回：成功返回 true，失败抛异常 
 */

payCoin
~~~~~~~~

**函数描述：**

``payCoin`` 函数用于转账资产。

**函数调用：**

::

 payCoin(address, amount[, input]);


**参数说明：**

address：发送 BU 的目标地址。

amount：发送 BU 的数量。

input：可选，合约参数，默认为空字符串。


**示例：**

.. code:: javascript

 payCoin("buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY", "10000", "{}");
 /*
 权限：可写
 返回：成功返回 true，失败抛异常 
 */

assert
~~~~~~~

**函数描述：**

``assert`` 函数用于断言验证。

**函数调用：**

::

 assert(condition[, message]);


**参数说明：**

condition：断言变量。

message：可选，失败时抛出异常的消息。


**示例：**

.. code:: javascript

 assert(1===1, "Not valid");
 /*
 权限：只读
 返回：成功返回 true，失败抛异常 
 */


内置变量
--------

本章节介绍智能合约开发过程涉及的一些内置变量，包括 `thisAddress`_、 `thisPayCoinAmount`_、 `thisPayAsset`_、 `blockNumber`_、 `blockTimestamp`_、 `sender`_、 `triggerIndex`_。

thisAddress
^^^^^^^^^^^^

**变量描述：**

全局变量 **thisAddress** 的值等于该合约账号的地址。
例如，账号 x 发起了一笔交易调用合约 Y ，本次执行过程中，thisAddress 的值就是 Y 合约账号的地址。



**示例代码：**


.. code:: JavaScript

::

 let bar = thisAddress; /* bar的值是Y合约的账号地址。 */



thisPayCoinAmount
^^^^^^^^^^^^^^^^^^^

**变量描述：**

本次支付操作的 BU Coin。



thisPayAsset
^^^^^^^^^^^^^^

**变量描述：**

本次支付操作的 Asset，为对象类型

::

 {"amount": 1000, "key" : {"issuer": "buQsZNDpqHJZ4g5hz47CqVMk5154w1bHKsHY", "code":"CNY"}}。

blockNumber
^^^^^^^^^^^^

**变量描述：**

当前区块高度。              


blockTimestamp
^^^^^^^^^^^^^^^^

**变量描述：**

当前区块时间戳。     


sender
^^^^^^^

**变量描述：**

调用者的地址。sender 的值为本次调用该合约的账号。
例如，某账号发起了一笔交易，该交易中某个操作是调用合约 Y（该操作的 source_address 是 x），那么在合约 Y 的执行过程中，sender 的值就是 x 账号的地址。


**示例代码：**

.. code:: JavaScript

 let bar = sender; /* 那么bar的值是x的账号地址。 */

triggerIndex
^^^^^^^^^^^^^^

triggerIndex 的值为触发本次合约的操作的序号。例如，某账号 A 发起了一笔交易 tx0，tx0 中第 0（从 0 开始计数）个操作是给某个合约账户转移资产（调用合约），那么 triggerIndex 的值就是 0。

**示例代码：**

::

 let bar = triggerIndex; /* bar 是一个非负整数*/


异常处理
--------

JavaScript 异常
^^^^^^^^^^^^^^^

当合约运行中出现未捕获的 JavaScript 异常时，做以下处理：

- 本次合约执行失败，合约中所有交易都不会生效。

- 触发本次合约的交易为失败。错误代码为 151。

执行交易失败
^^^^^^^^^^^^

合约中可以执行多个交易，只要有一个交易失败，就会抛出异常，导致整个交易失败。

示例
-----

本章节介绍了三个基于Java语言的智能合约开发实例场景，其中场景一和场景二是相关联的。实例场景都是基于以下遵循 CTP 1.0 协议的智能合约代码，
该代码来自 `contractBasedToken.js <https://github.com/bumoproject/bumo/blob/master/src/ledger/contractBasedToken.js>`_ 。

.. code:: javascript
 
 /*
 Contract-based token template
 OBSERVING CTP 1.0
 
 STATEMENT:
 Any organizations or individuals that intend to issue contract-based tokens on BuChain should abide by the Contract-based Token Protocol(CTP). Therefore, any contract that 
 created on BuChain including global attributes of CTP, we treat it as contract-based token.
 */

 'use strict';
 let globalAttribute = {};
 function globalAttributeKey(){
 return 'global_attribute';
 }

 function loadGlobalAttribute(){
 if(Object.keys(globalAttribute).length === 0){
 let value = storageLoad(globalAttributeKey());
 assert(value !== false, 'Get global attribute from metadata failed.');
 globalAttribute = JSON.parse(value);
 }
 }

 function storeGlobalAttribute(){
 let value = JSON.stringify(globalAttribute);
 storageStore(globalAttributeKey(), value);
 }

 function powerOfBase10(exponent){
 let i = 0;
 let power = 1;
 while(i < exponent){
 power = power * 10;
 i = i + 1;
 }
 return power;
 }

 function makeBalanceKey(address){
 return 'balance_' + address;
 }
 function makeAllowanceKey(owner, spender){
 return 'allow_' + owner + '_to_' + spender;
 }

 function valueCheck(value) {
 if (value.startsWith('-') || value === '0') {
 return false;
 }
 return true;
 }

 function approve(spender, value){
 assert(addressCheck(spender) === true, 'Arg-spender is not a valid address.');
 assert(stoI64Check(value) === true, 'Arg-value must be alphanumeric.');
 assert(valueCheck(value) === true, 'Arg-value must be positive number.');

 let key = makeAllowanceKey(sender, spender);
 storageStore(key, value);
 tlog('approve', sender, spender, value);
 return true;
 }

 function allowance(owner, spender){
 assert(addressCheck(owner) === true, 'Arg-owner is not a valid address.');
 assert(addressCheck(spender) === true, 'Arg-spender is not a valid address.');
 
 let key = makeAllowanceKey(owner, spender);
 let value = storageLoad(key);
 assert(value !== false, 'Get allowance ' + owner + ' to ' + spender + ' from metadata failed.');

  return value;
 }

 function transfer(to, value){
 assert(addressCheck(to) === true, 'Arg-to is not a valid address.');
 assert(stoI64Check(value) === true, 'Arg-value must be alphanumeric.');
 assert(valueCheck(value)  === true, 'Arg-value must be positive number.');
 if(sender === to) {
 tlog('transfer', sender, to, value); 
 return true;
 }

 let senderKey = makeBalanceKey(sender);
 let senderValue = storageLoad(senderKey);
 assert(senderValue !== false, 'Get balance of ' + sender + ' from metadata failed.');

 assert(int64Compare(senderValue, value) >= 0, 'Balance:' + senderValue + ' of sender:' + sender + ' < transfer value:' + value + '.');

 let toKey = makeBalanceKey(to);
 let toValue = storageLoad(toKey);
 toValue = (toValue === false) ? value : int64Add(toValue, value); 
 storageStore(toKey, toValue);

 senderValue = int64Sub(senderValue, value);
 storageStore(senderKey, senderValue);
 tlog('transfer', sender, to, value);
 return true;
 }

 function assign(to, value){ 
    assert(addressCheck(to) === true, 'Arg-to is not a valid address.'); 
    assert(stoI64Check(value) === true, 'Arg-value must be alphanumeric.'); 
    assert(valueCheck(value) === true, 'Arg-value must be positive number.'); 
     
    if(thisAddress === to) { 
        tlog('assign', to, value); 
        return true; 
        } 
     
    loadGlobalAttribute(); 
    assert(sender === globalAttribute.contractOwner, sender + ' has no permission to assign contract balance.'); 
    assert(int64Compare(globalAttribute.balance, value) >= 0, 'Balance of contract:' + globalAttribute.balance + ' < assign value:' + value + '.'); 
 
    let toKey = makeBalanceKey(to); 
    let toValue = storageLoad(toKey); 
    toValue = (toValue === false) ? value : int64Add(toValue, value);  
    storageStore(toKey, toValue); 
 
    globalAttribute.balance = int64Sub(globalAttribute.balance, value); 
    storeGlobalAttribute(); 
 
    tlog('assign', to, value); 
 
    return true; 
 } 
 function transferFrom(from, to, value){ 
    assert(addressCheck(from) === true, 'Arg-from is not a valid address.'); 
    assert(addressCheck(to) === true, 'Arg-to is not a valid address.'); 
    assert(stoI64Check(value) === true, 'Arg-value must be alphanumeric.'); 
    assert(valueCheck(value) === true, 'Arg-value must be positive number.'); 
     
    if(from === to) { 
        tlog('transferFrom', sender, from, to, value); 
        return true; 
    } 
     
    let fromKey = makeBalanceKey(from); 
    let fromValue = storageLoad(fromKey); 
    assert(fromValue !== false, 'Get value failed, maybe ' + from + ' has no value.'); 
    assert(int64Compare(fromValue, value) >= 0, from + ' balance:' + fromValue + ' < transfer value:' + value + '.'); 
 
    let allowValue = allowance(from, sender); 
    assert(int64Compare(allowValue, value) >= 0, 'Allowance value:' + allowValue + ' < transfer value:' + value + ' from ' + from + ' to ' + to  + '.'); 
 
    let toKey = makeBalanceKey(to); 
    let toValue = storageLoad(toKey); 
    toValue = (toValue === false) ? value : int64Add(toValue, value); 
    storageStore(toKey, toValue); 
 
    fromValue = int64Sub(fromValue, value); 
    storageStore(fromKey, fromValue); 
 
    let allowKey = makeAllowanceKey(from, sender); 
    allowValue   = int64Sub(allowValue, value); 
    storageStore(allowKey, allowValue); 
 
    tlog('transferFrom', sender, from, to, value); 
 
    return true; 
 } 
 
 function changeOwner(address){ 
    assert(addressCheck(address) === true, 'Arg-address is not a valid address.'); 
 
    loadGlobalAttribute(); 
    assert(sender === globalAttribute.contractOwner, sender + ' has no permission to modify contract ownership.'); 
 
    globalAttribute.contractOwner = address; 
    storeGlobalAttribute(); 
 
    tlog('changeOwner', sender, address); 
 } 
 
 function name() { 
    return globalAttribute.name; 
 } 
 
 function symbol(){ 
    return globalAttribute.symbol; 
 } 
 
 function decimals(){ 
    return globalAttribute.decimals; 
 } 
 
 function totalSupply(){ 
    return globalAttribute.totalSupply; 
 } 
 
 function ctp(){ 
 return globalAttribute.ctp; 
 } 
 
 function contractInfo(){ 
    return globalAttribute; 
 } 
 
 function balanceOf(address){ 
    assert(addressCheck(address) === true, 'Arg-address is not a valid address.'); 
 
    if(address === globalAttribute.contractOwner || address === thisAddress){ 
        return globalAttribute.balance; 
    } 
 
    let key = makeBalanceKey(address); 
    let value = storageLoad(key); 
    assert(value !== false, 'Get balance of ' + address + ' from metadata failed.'); 
 
    return value; 
 } 
 
 function init(input_str){ 
    let input = JSON.parse(input_str); 
 
    assert(stoI64Check(input.params.supply) === true && 
           typeof input.params.name === 'string' && 
           typeof input.params.symbol === 'string' && 
           typeof input.params.decimals === 'number', 
           'Args check failed.'); 
 
    globalAttribute.ctp = '1.0'; 
    globalAttribute.name = input.params.name; 
    globalAttribute.symbol = input.params.symbol; 
    globalAttribute.decimals = input.params.decimals; 
    globalAttribute.totalSupply = int64Mul(input.params.supply, powerOfBase10(globalAttribute.decimals)); 
    globalAttribute.contractOwner = sender; 
    globalAttribute.balance = globalAttribute.totalSupply; 
 
    storageStore(globalAttributeKey(), JSON.stringify(globalAttribute)); 
 } 
 
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
        throw '<unidentified operation type>'; 
    } 
 } 
 
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
    else if(input.method === 'ctp'){ 
        result.ctp = ctp(); 
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

      
实例场景一
^^^^^^^^^^^

某资方在 BuChain 上基于CTP 1.0发行代码为 CGO、名称为 Contract Global、总发行量为 10 亿的智能合约币种，具体信息如下：


+-------------------------+----------+------------------+---------------+
| 字段                    | 是否必填 | 示例             |     描述      |
+=========================+==========+==================+===============+
| name                    | 是       | Contract Global  | 币种名称      |
+-------------------------+----------+------------------+---------------+
| symbol                  | 是       | CGO              | 币种代码      |
+-------------------------+----------+------------------+---------------+
| totalSupply             | 是       | 1000000000       | 资产总发行量  |
+-------------------------+----------+------------------+---------------+
| decimals                | 是       | 8                | 币种精度      |
+-------------------------+----------+------------------+---------------+
| ctp                     | 是       |  1.0             | 协议版本号    |
+-------------------------+----------+------------------+---------------+

线上demo请看: `CreateContractDemo.java <https://github.com/bumoproject/bumo-sdk-java/blob/develop/examples/src/main/java/io/bumo/sdk/example/CreateContractDemo.java>`_ 。

本场景的具体执行过程包括 `验证代码是否有效`_、`文本压缩`_、:ref:`创建SDK实例-1`、`创建资方账户`_、`激活资方账户`_、:ref:`获取资方账户的序列号-1`、`组装创建合约账户并发行CGO代币操作`_、
:ref:`序列化交易-1`、:ref:`签名交易-1`、:ref:`发送交易-1`、:ref:`查询交易是否执行成功-1`。






验证代码是否有效
~~~~~~~~~~~~~~~~

打开在线检测页面: http://bumo.chinacloudapp.cn:36002/jslint.html ，将上面的智能合约代码拷贝到编辑框中，点击 **JSLint** 按钮，这里提示智能合约代码没有问题。 
如果出现背景是红色的 warning 提示，表示语法有问题，如下图：

|warnings|

如果没有语法问题，弹出的提示如下图：

|nowarnings|

文本压缩
~~~~~~~~

打开在线文本压缩页面: https://jsmin.51240.com/ ，将验证无误的智能合约代码拷贝到页面中的编辑框中，然后点击 **压缩** 按钮，将压缩后的字符串拷贝下来，如下图：

|compressedString|


.. _创建SDK实例-1:

创建SDK实例
~~~~~~~~~~~~

创建实例并设置 url (部署的某节点的IP和端口)。 

环境说明：

+-------------------------+--------------------+------------------+----------------------------------+
| 网络环境                | IP                 | Port             | 区块链浏览器                     |
+=========================+====================+==================+==================================+
| 主网                    | seed1.bumo.io      | 16002            | https://explorer.bumo.io         |
+-------------------------+--------------------+------------------+----------------------------------+
| 测试                    | seed1.bumotest.io  | 26002            | http://explorer.bumotest.io      |
+-------------------------+--------------------+------------------+----------------------------------+


代码示例：

.. code:: javascript

 String url = "http://seed1.bumotest.io:26002"; 
 SDK sdk = SDK.getInstance(url); 
 
在 BuChain 网络里，每个区块产生的时间是 10 秒，每个交易只需要一次确认即可得到交易终态。


创建资方账户
~~~~~~~~~~~~

创建资方账户的代码如下：

.. code:: javascript

 public static AccountCreateResult createAccount() { 
    AccountCreateResponse response = sdk.getAccountService().create(); 
    if (response.getErrorCode() != 0) { 
        return null; 
    } 
    return response.getResult(); 
 }

创建账户的返回值如下：

::

 AccountCreateResult 
   address: buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH 
   privateKey: privbs4iBCugQeb2eiycU8RzqkPqd28eaAYrRJGwtJTG8FVHjwAyjiyC 
 publicKey: b00135e99d67a4c2e10527f766e08bc6afd4420951628149042fdad6584a5321c23c716a528b

.. note::
 
 通过该方式创建的账户是未被激活的账户。


激活资方账户
~~~~~~~~~~~~

账户未被激活时需要通过已被激活（已上链）的账户进行激活。已被激活的资方账户请跳过本节内容。


.. note:: - 主网环境：账户激活可以通过小布口袋（钱包）给该资方账户转 10.09 BU（用于支付资产发行时需要的交易费用），即可激活该账户。

       - 测试环境：资方向 gavin@bumo.io 发出申请，申请内容是资产的账户地址。


.. _获取资方账户的序列号-1:

获取资方账户的序列号
~~~~~~~~~~~~~~~~~~~

每个账户都维护着自己的序列号，该序列号从1开始，依次递增，一个序列号标志着一个该账户的交易。获取资方账号序列号的代码如下：

::

 public long getAccountNonce() {
 long nonce = 0;

    // Init request
    String accountAddress = [资方账户地址];
    AccountGetNonceRequest request = new AccountGetNonceRequest();
    request.setAddress(accountAddress);

    // Call getNonce
    AccountGetNonceResponse response = sdk.getAccountService().getNonce(request);
    if (0 == response.getErrorCode()) {
        nonce = response.getResult().getNonce();
    } else {
        System.out.println("error: " + response.getErrorDesc());
 }
 return nonce;
 }

.. note::
 如果查询不到某账户，则表示该账户未激活。


返回值如下：

::

 nonce: 0

组装创建合约账户并发行CGO代币操作
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

代码中将压缩好的合约代码赋值给 payload 变量，具体代码如下：

.. code:: javascript
 
 public BaseOperation[] buildOperations() { 
 // The account address to issue apt1.0 token 
 String createContractAddress = "buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH"; 
 // Contract account initialization BU，the unit is MO，and 1 BU = 10^8 MO 
 Long initBalance = ToBaseUnit.BU2MO("0.01"); 
 // The token name 
    String name = "Contract Global"; 
    // The token code 
    String symbol = "CGO"; 
    // The token total supply number 
    Long supply = 1000000000L; 
    // The token decimals 
 Integer decimals = 8; 
 // Contract code 
 String payload = "'use strict';
 let globalAttribute={};
 
 function globalAttributeKey()
 {return'global_attribute';}

 function loadGlobalAttribute()
 {if(Object.keys(globalAttribute).length===0)
 {let value=storageLoad(globalAttributeKey());
 assert(value!==false,'Get global attribute from metadata failed.');
 globalAttribute=JSON.parse(value);}}
 
 function storeGlobalAttribute()
 {let value=JSON.stringify(globalAttribute);
 storageStore(globalAttributeKey(),value);}
 
 function powerOfBase10(exponent)
 {let i=0;let power=1;while(i<exponent)
 {power=power*10;i=i+1;}return power;}
 
 function makeBalanceKey(address)
 {return'balance_'+address;}
 
 function makeAllowanceKey(owner,spender)
 {return'allow_'+owner+'_to_'+spender;}
 
 function valueCheck(value)
 {if(value.startsWith('-')||value==='0')
 {return false;}return true;}
 
 function approve(spender,value)
 {assert(addressCheck(spender)===true,'Arg-spender is not a valid address.');
 assert(stoI64Check(value)===true,'Arg-value must be alphanumeric.');
 assert(valueCheck(value)===true,'Arg-value must be positive number.');
 let key=makeAllowanceKey(sender,spender);
 storageStore(key,value);
 tlog('approve',sender,spender,value);return true;}

 function allowance(owner,spender)
 {assert(addressCheck(owner)===true,'Arg-owner is not a valid address.');
 assert(addressCheck(spender)===true,'Arg-spender is not a valid address.');
 let key=makeAllowanceKey(owner,spender);
 let value=storageLoad(key);
 assert(value!==false,'Get allowance '+owner+' to '+spender+' from metadata failed.');
 return value;}
 
 function transfer(to,value)
 {assert(addressCheck(to)===true,'Arg-to is not a valid address.');
 assert(stoI64Check(value)===true,'Arg-value must be alphanumeric.');
 assert(valueCheck(value)===true,'Arg-value must be positive number.');
 if(sender===to)
 {tlog('transfer',sender,to,value);
 return true;}
 let senderKey=makeBalanceKey(sender);
 let senderValue=storageLoad(senderKey);
 assert(senderValue!==false,'Get balance of '+sender+' from metadata failed.');
 assert(int64Compare(senderValue,value)>=0,'Balance:'+senderValue+' of sender:'+sender+' < transfer value:'+value+'.');
 let toKey=makeBalanceKey(to);
 let toValue=storageLoad(toKey);
 toValue=(toValue===false)?value:int64Add(toValue,value);
 storageStore(toKey,toValue);
 senderValue=int64Sub(senderValue,value);
 storageStore(senderKey,senderValue);
 tlog('transfer',sender,to,value);
 return true;}
 
 function assign(to,value)
 {assert(addressCheck(to)===true,'Arg-to is not a valid address.');
 assert(stoI64Check(value)===true,'Arg-value must be alphanumeric.');
 assert(valueCheck(value)===true,'Arg-value must be positive number.');
 if(thisAddress===to){tlog('assign',to,value);return true;}
 loadGlobalAttribute();
 assert(sender===globalAttribute.contractOwner,sender+' has no permission to assign contract balance.');
 assert(int64Compare(globalAttribute.balance,value)>=0,'Balance of contract:'+globalAttribute.balance+' < assign value:'+value+'.');
 let toKey=makeBalanceKey(to);
 let toValue=storageLoad(toKey);
 toValue=(toValue===false)?value:int64Add(toValue,value);
 storageStore(toKey,toValue);
 globalAttribute.balance=int64Sub(globalAttribute.balance,value);
 storeGlobalAttribute();
 tlog('assign',to,value);
 return true;}
 
 function transferFrom(from,to,value)
 {assert(addressCheck(from)===true,'Arg-from is not a valid address.');
 assert(addressCheck(to)===true,'Arg-to is not a valid address.');
 assert(stoI64Check(value)===true,'Arg-value must be alphanumeric.');
 assert(valueCheck(value)===true,'Arg-value must be positive number.');
 if(from===to){tlog('transferFrom',sender,from,to,value);return true;}
 let fromKey=makeBalanceKey(from);
 let fromValue=storageLoad(fromKey);
 assert(fromValue!==false,'Get value failed, maybe '+from+' has no value.');
 assert(int64Compare(fromValue,value)>=0,from+' balance:'+fromValue+' < transfer value:'+value+'.');
 let allowValue=allowance(from,sender);
 assert(int64Compare(allowValue,value)>=0,'Allowance value:'+allowValue+' < transfer value:'+value+' from '+from+' to '+to+'.');
 let toKey=makeBalanceKey(to);
 let toValue=storageLoad(toKey);
 toValue=(toValue===false)?value:int64Add(toValue,value);
 storageStore(toKey,toValue);
 fromValue=int64Sub(fromValue,value);
 storageStore(fromKey,fromValue);
 let allowKey=makeAllowanceKey(from,sender);
 allowValue=int64Sub(allowValue,value);
 storageStore(allowKey,allowValue);
 tlog('transferFrom',sender,from,to,value);
 return true;}

 function changeOwner(address)
 {assert(addressCheck(address)===true,'Arg-address is not a valid address.');
 loadGlobalAttribute();
 assert(sender===globalAttribute.contractOwner,sender+' has no permission to modify contract ownership.');
 globalAttribute.contractOwner=address;storeGlobalAttribute();
 tlog('changeOwner',sender,address);}
 
 function name()
 {return globalAttribute.name;}
 
 function symbol()
 {return globalAttribute.symbol;}
 
 function decimals()
 {return globalAttribute.decimals;}
 
 function totalSupply()
 {return globalAttribute.totalSupply;}
 
 function ctp()
 {return globalAttribute.ctp;}
 
 function contractInfo()
 {return globalAttribute;}
 
 function balanceOf(address)
 {assert(addressCheck(address)===true,'Arg-address is not a valid address.');
 if(address===globalAttribute.contractOwner||address===thisAddress)
 {return globalAttribute.balance;}
 let key=makeBalanceKey(address);
 let value=storageLoad(key);
 assert(value!==false,'Get balance of '+address+' from metadata failed.');
 return value;}
 
 function init(input_str)
 {let input=JSON.parse(input_str);
 assert(stoI64Check(input.params.supply)===true&&typeof input.params.name==='string'&&typeof input.params.symbol==='string'&&typeof input.params.decimals==='number','Args check failed.');
 globalAttribute.ctp='1.0';
 globalAttribute.name=input.params.name;
 globalAttribute.symbol=input.params.symbol;
 globalAttribute.decimals=input.params.decimals;
 globalAttribute.totalSupply=int64Mul(input.params.supply,powerOfBase10(globalAttribute.decimals));
 globalAttribute.contractOwner=sender;
 globalAttribute.balance=globalAttribute.totalSupply;
 storageStore(globalAttributeKey(),JSON.stringify(globalAttribute));}
 
 function main(input_str){let input=JSON.parse(input_str);
 if(input.method==='transfer')
 {transfer(input.params.to,input.params.value);}
 else 
 if(input.method==='transferFrom')
 {transferFrom(input.params.from,input.params.to,input.params.value);}
 else
 if(input.method==='approve')
 {approve(input.params.spender,input.params.value);}
 else 
 if(input.method==='assign')
 {assign(input.params.to,input.params.value);}
 else 
 if(input.method==='changeOwner')
 {changeOwner(input.params.address);}
 else{throw'<unidentified operation type>';}}
 
 function query(input_str)
 {loadGlobalAttribute();
 let result={};
 let input=JSON.parse(input_str);
 if(input.method==='name')
 {result.name=name();}
 else 
 if(input.method==='symbol')
 {result.symbol=symbol();}
 else 
 if(input.method==='decimals')
 {result.decimals=decimals();}
 else 
 if(input.method==='totalSupply')
 {result.totalSupply=totalSupply();}
 else 
 if(input.method==='ctp')
 {result.ctp=ctp();}
 else 
 if(input.method==='contractInfo')
 {result.contractInfo=contractInfo();}
 else 
 if(input.method==='balanceOf')
 {result.balance=balanceOf(input.params.address);}
 else 
 if(input.method==='allowance')
 {result.allowance=allowance(input.params.owner,input.params.spender);}
 else
 {throw'<unidentified operation type>';}
 log(result);return JSON.stringify(result);}"; 
 
 // Init initInput 
 JSONObject initInput = new JSONObject(); 
 JSONObject params = new JSONObject(); 
 params.put("name", name); 
 params.put("symbol", symbol); 
 params.put("decimals", decimals); 
 params.put("supply", supply); 
 initInput.put("params", params);  
 
 // Build create contract operation 
 ContractCreateOperation contractCreateOperation = new ContractCreateOperation(); 
 contractCreateOperation.setSourceAddress(createContractAddress); 
 contractCreateOperation.setInitBalance(initBalance); 
 contractCreateOperation.setPayload(payload); 
 contractCreateOperation.setInitInput(initInput.toJSONString()); 
 contractCreateOperation.setMetadata("create ctp 1.0 contract"); 
     
 BaseOperation[] operations = { contractCreateOperation }; 
 return operations; 
 } 

.. _序列化交易-1:

序列化交易
~~~~~~~~~~~


序列化交易即将交易序列化以便网络传输。


.. note:: - feeLimit: 本次交易发起方最多支付本次交易的交易费用，发行资产操作请填写10.08BU。

       - nonce: 本次交易发起方的交易序列号，该值由当前账户的 nonce 值加1得到。



序列化交易的具体代码如下，示例中的参数 nonce 是调用 getAccountNonce 得到的账户序列号，参数 operations 是调用 buildOperations 得到发行资产的操作。


.. code:: javascript

 public String seralizeTransaction(Long nonce,  BaseOperation[] operations) { 
 String transactionBlob = null; 
 
 // The account address to create contract and issue ctp 1.0 token 
 String senderAddresss = "buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH"; 
    // The gasPrice is fixed at 1000L, the unit is MO 
    Long gasPrice = 1000L; 
    // Set up the maximum cost 10.08BU 
    Long feeLimit = ToBaseUnit.BU2MO("10.08"); 
    // Nonce should add 1 
 nonce += 1; 
 
 // Build transaction  Blob 
 TransactionBuildBlobRequest transactionBuildBlobRequest = new TransactionBuildBlobRequest(); 
 transactionBuildBlobRequest.setSourceAddress(senderAddresss); 
 transactionBuildBlobRequest.setNonce(nonce); 
 transactionBuildBlobRequest.setFeeLimit(feeLimit); 
 transactionBuildBlobRequest.setGasPrice(gasPrice); 
 for (int i = 0; i < operations.length; i++) { 
    transactionBuildBlobRequest.addOperation(operations[i]); 
 } 
 TransactionBuildBlobResponse transactionBuildBlobResponse = sdk.getTransactionService().buildBlob(transactionBuildBlobRequest); 
 if (transactionBuildBlobResponse.getErrorCode() == 0) { 
 transactionBlob = transactionBuildBlobResponse. getResult().getTransactionBlob(); 
 } else { 
    System.out.println("error: " + transactionBuildBlobResponse.getErrorDesc()); 
 } 
 return transactionBlob; 
 } 

序列化交易的返回值如下：

::
 
 transactionBlob: 
 0A24627551594C745271346A336571626A564E47596B4B596F33734C42715733545148
 32784810011880B8D3E00320E8073AA23908011224627551594C745271346A33657162
 6A564E47596B4B596F33734C427157335451483278481A176372656174652063747020
 312E3020636F6E747261637422DE3812F83712F5372775736520737472696374273B6C
 657420676C6F62616C4174747269627574653D7B7D3B66756E6374696F6E20676C6F62
 616C4174747269627574654B657928297B72657475726E27676C6F62616C5F61747472
 6962757465273B7D66756E6374696F6E206C6F6164476C6F62616C4174747269627574
 6528297B6966284F626A6563742E6B65797328676C6F62616C41747472696275746529
 2E6C656E6774683D3D3D30297B6C65742076616C75653D73746F726167654C6F616428
 676C6F62616C4174747269627574654B65792829293B6173736572742876616C756521
 3D3D66616C73652C2747657420676C6F62616C206174747269627574652066726F6D20
 6D65746164617461206661696C65642E27293B676C6F62616C4174747269627574653D
 4A534F4E2E70617273652876616C7565293B7D7D66756E6374696F6E2073746F726547
 6C6F62616C41747472696275746528297B6C65742076616C75653D4A534F4E2E737472
 696E6769667928676C6F62616C417474726962757465293B73746F7261676553746F72
 6528676C6F62616C4174747269627574654B657928292C76616C7565293B7D66756E63
 74696F6E20706F7765724F66426173653130286578706F6E656E74297B6C657420693D
 303B6C657420706F7765723D313B7768696C6528693C6578706F6E656E74297B706F77
 65723D706F7765722A31303B693D692B313B7D72657475726E20706F7765723B7D6675
 6E6374696F6E206D616B6542616C616E63654B65792861646472657373297B72657475
 726E2762616C616E63655F272B616464726573733B7D66756E6374696F6E206D616B65
 416C6C6F77616E63654B6579286F776E65722C7370656E646572297B72657475726E27
 616C6C6F775F272B6F776E65722B275F746F5F272B7370656E6465723B7D66756E6374
 696F6E2076616C7565436865636B2876616C7565297B69662876616C75652E73746172
 74735769746828272D27297C7C76616C75653D3D3D273027297B72657475726E206661
 6C73653B7D72657475726E20747275653B7D66756E6374696F6E20617070726F766528
 7370656E6465722C76616C7565297B6173736572742861646472657373436865636B28
 7370656E646572293D3D3D747275652C274172672D7370656E646572206973206E6F74
 20612076616C696420616464726573732E27293B6173736572742873746F4936344368
 65636B2876616C7565293D3D3D747275652C274172672D76616C7565206D7573742062
 6520616C7068616E756D657269632E27293B6173736572742876616C7565436865636B
 2876616C7565293D3D3D747275652C274172672D76616C7565206D7573742062652070
 6F736974697665206E756D6265722E27293B6C6574206B65793D6D616B65416C6C6F77
 616E63654B65792873656E6465722C7370656E646572293B73746F7261676553746F72
 65286B65792C76616C7565293B746C6F672827617070726F7665272C73656E6465722C
 7370656E6465722C76616C7565293B72657475726E20747275653B7D66756E6374696F
 6E20616C6C6F77616E6365286F776E65722C7370656E646572297B6173736572742861
 646472657373436865636B286F776E6572293D3D3D747275652C274172672D6F776E65
 72206973206E6F7420612076616C696420616464726573732E27293B61737365727428
 61646472657373436865636B287370656E646572293D3D3D747275652C274172672D73
 70656E646572206973206E6F7420612076616C696420616464726573732E27293B6C65
 74206B65793D6D616B65416C6C6F77616E63654B6579286F776E65722C7370656E6465
 72293B6C65742076616C75653D73746F726167654C6F6164286B6579293B6173736572
 742876616C7565213D3D66616C73652C2747657420616C6C6F77616E636520272B6F77
 6E65722B2720746F20272B7370656E6465722B272066726F6D206D6574616461746120
 6661696C65642E27293B72657475726E2076616C75653B7D66756E6374696F6E207472
 616E7366657228746F2C76616C7565297B617373657274286164647265737343686563
 6B28746F293D3D3D747275652C274172672D746F206973206E6F7420612076616C6964
 20616464726573732E27293B6173736572742873746F493634436865636B2876616C75
 65293D3D3D747275652C274172672D76616C7565206D75737420626520616C7068616E
 756D657269632E27293B6173736572742876616C7565436865636B2876616C7565293D
 3D3D747275652C274172672D76616C7565206D75737420626520706F73697469766520
 6E756D6265722E27293B69662873656E6465723D3D3D746F297B746C6F672827747261
 6E73666572272C73656E6465722C746F2C76616C7565293B72657475726E2074727565
 3B7D6C65742073656E6465724B65793D6D616B6542616C616E63654B65792873656E64
 6572293B6C65742073656E64657256616C75653D73746F726167654C6F61642873656E
 6465724B6579293B6173736572742873656E64657256616C7565213D3D66616C73652C
 274765742062616C616E6365206F6620272B73656E6465722B272066726F6D206D6574
 6164617461206661696C65642E27293B61737365727428696E743634436F6D70617265
 2873656E64657256616C75652C76616C7565293E3D302C2742616C616E63653A272B73
 656E64657256616C75652B27206F662073656E6465723A272B73656E6465722B27203C
 207472616E736665722076616C75653A272B76616C75652B272E27293B6C657420746F
 4B65793D6D616B6542616C616E63654B657928746F293B6C657420746F56616C75653D
 73746F726167654C6F616428746F4B6579293B746F56616C75653D28746F56616C7565
 3D3D3D66616C7365293F76616C75653A696E74363441646428746F56616C75652C7661
 6C7565293B73746F7261676553746F726528746F4B65792C746F56616C7565293B7365
 6E64657256616C75653D696E7436345375622873656E64657256616C75652C76616C75
 65293B73746F7261676553746F72652873656E6465724B65792C73656E64657256616C
 7565293B746C6F6728277472616E73666572272C73656E6465722C746F2C76616C7565
 293B72657475726E20747275653B7D66756E6374696F6E2061737369676E28746F2C76
 616C7565297B6173736572742861646472657373436865636B28746F293D3D3D747275
 652C274172672D746F206973206E6F7420612076616C696420616464726573732E2729
 3B6173736572742873746F493634436865636B2876616C7565293D3D3D747275652C27
 4172672D76616C7565206D75737420626520616C7068616E756D657269632E27293B61
 73736572742876616C7565436865636B2876616C7565293D3D3D747275652C27417267
 2D76616C7565206D75737420626520706F736974697665206E756D6265722E27293B69
 662874686973416464726573733D3D3D746F297B746C6F67282761737369676E272C74
 6F2C76616C7565293B72657475726E20747275653B7D6C6F6164476C6F62616C417474
 72696275746528293B6173736572742873656E6465723D3D3D676C6F62616C41747472
 69627574652E636F6E74726163744F776E65722C73656E6465722B2720686173206E6F
 207065726D697373696F6E20746F2061737369676E20636F6E74726163742062616C61
 6E63652E27293B61737365727428696E743634436F6D7061726528676C6F62616C4174
 747269627574652E62616C616E63652C76616C7565293E3D302C2742616C616E636520
 6F6620636F6E74726163743A272B676C6F62616C4174747269627574652E62616C616E
 63652B27203C2061737369676E2076616C75653A272B76616C75652B272E27293B6C65
 7420746F4B65793D6D616B6542616C616E63654B657928746F293B6C657420746F5661
 6C75653D73746F726167654C6F616428746F4B6579293B746F56616C75653D28746F56
 616C75653D3D3D66616C7365293F76616C75653A696E74363441646428746F56616C75
 652C76616C7565293B73746F7261676553746F726528746F4B65792C746F56616C7565
 293B676C6F62616C4174747269627574652E62616C616E63653D696E74363453756228
 676C6F62616C4174747269627574652E62616C616E63652C76616C7565293B73746F72
 65476C6F62616C41747472696275746528293B746C6F67282761737369676E272C746F
 2C76616C7565293B72657475726E20747275653B7D66756E6374696F6E207472616E73
 66657246726F6D2866726F6D2C746F2C76616C7565297B617373657274286164647265
 7373436865636B2866726F6D293D3D3D747275652C274172672D66726F6D206973206E
 6F7420612076616C696420616464726573732E27293B61737365727428616464726573
 73436865636B28746F293D3D3D747275652C274172672D746F206973206E6F74206120
 76616C696420616464726573732E27293B6173736572742873746F493634436865636B
 2876616C7565293D3D3D747275652C274172672D76616C7565206D7573742062652061
 6C7068616E756D657269632E27293B6173736572742876616C7565436865636B287661
 6C7565293D3D3D747275652C274172672D76616C7565206D75737420626520706F7369
 74697665206E756D6265722E27293B69662866726F6D3D3D3D746F297B746C6F672827
 7472616E7366657246726F6D272C73656E6465722C66726F6D2C746F2C76616C756529
 3B72657475726E20747275653B7D6C65742066726F6D4B65793D6D616B6542616C616E
 63654B65792866726F6D293B6C65742066726F6D56616C75653D73746F726167654C6F
 61642866726F6D4B6579293B6173736572742866726F6D56616C7565213D3D66616C73
 652C274765742076616C7565206661696C65642C206D6179626520272B66726F6D2B27
 20686173206E6F2076616C75652E27293B61737365727428696E743634436F6D706172
 652866726F6D56616C75652C76616C7565293E3D302C66726F6D2B272062616C616E63
 653A272B66726F6D56616C75652B27203C207472616E736665722076616C75653A272B
 76616C75652B272E27293B6C657420616C6C6F7756616C75653D616C6C6F77616E6365
 2866726F6D2C73656E646572293B61737365727428696E743634436F6D706172652861
 6C6C6F7756616C75652C76616C7565293E3D302C27416C6C6F77616E63652076616C75
 653A272B616C6C6F7756616C75652B27203C207472616E736665722076616C75653A27
 2B76616C75652B272066726F6D20272B66726F6D2B2720746F20272B746F2B272E2729
 3B6C657420746F4B65793D6D616B6542616C616E63654B657928746F293B6C65742074
 6F56616C75653D73746F726167654C6F616428746F4B6579293B746F56616C75653D28
 746F56616C75653D3D3D66616C7365293F76616C75653A696E74363441646428746F56
 616C75652C76616C7565293B73746F7261676553746F726528746F4B65792C746F5661
 6C7565293B66726F6D56616C75653D696E7436345375622866726F6D56616C75652C76
 616C7565293B73746F7261676553746F72652866726F6D4B65792C66726F6D56616C75
 65293B6C657420616C6C6F774B65793D6D616B65416C6C6F77616E63654B6579286672
 6F6D2C73656E646572293B616C6C6F7756616C75653D696E74363453756228616C6C6F
 7756616C75652C76616C7565293B73746F7261676553746F726528616C6C6F774B6579
 2C616C6C6F7756616C7565293B746C6F6728277472616E7366657246726F6D272C7365
 6E6465722C66726F6D2C746F2C76616C7565293B72657475726E20747275653B7D6675
 6E6374696F6E206368616E67654F776E65722861646472657373297B61737365727428
 61646472657373436865636B2861646472657373293D3D3D747275652C274172672D61
 646472657373206973206E6F7420612076616C696420616464726573732E27293B6C6F
 6164476C6F62616C41747472696275746528293B6173736572742873656E6465723D3D
 3D676C6F62616C4174747269627574652E636F6E74726163744F776E65722C73656E64
 65722B2720686173206E6F207065726D697373696F6E20746F206D6F6469667920636F
 6E7472616374206F776E6572736869702E27293B676C6F62616C417474726962757465
 2E636F6E74726163744F776E65723D616464726573733B73746F7265476C6F62616C41
 747472696275746528293B746C6F6728276368616E67654F776E6572272C73656E6465
 722C61646472657373293B7D66756E6374696F6E206E616D6528297B72657475726E20
 676C6F62616C4174747269627574652E6E616D653B7D66756E6374696F6E2073796D62
 6F6C28297B72657475726E20676C6F62616C4174747269627574652E73796D626F6C3B
 7D66756E6374696F6E20646563696D616C7328297B72657475726E20676C6F62616C41
 74747269627574652E646563696D616C733B7D66756E6374696F6E20746F74616C5375
 70706C7928297B72657475726E20676C6F62616C4174747269627574652E746F74616C
 537570706C793B7D66756E6374696F6E2063747028297B72657475726E20676C6F6261
 6C4174747269627574652E6374703B7D66756E6374696F6E20636F6E7472616374496E
 666F28297B72657475726E20676C6F62616C4174747269627574653B7D66756E637469
 6F6E2062616C616E63654F662861646472657373297B61737365727428616464726573
 73436865636B2861646472657373293D3D3D747275652C274172672D61646472657373
 206973206E6F7420612076616C696420616464726573732E27293B6966286164647265
 73733D3D3D676C6F62616C4174747269627574652E636F6E74726163744F776E65727C
 7C616464726573733D3D3D7468697341646472657373297B72657475726E20676C6F62
 616C4174747269627574652E62616C616E63653B7D6C6574206B65793D6D616B654261
 6C616E63654B65792861646472657373293B6C65742076616C75653D73746F72616765
 4C6F6164286B6579293B6173736572742876616C7565213D3D66616C73652C27476574
 2062616C616E6365206F6620272B616464726573732B272066726F6D206D6574616461
 7461206661696C65642E27293B72657475726E2076616C75653B7D66756E6374696F6E
 20696E697428696E7075745F737472297B6C657420696E7075743D4A534F4E2E706172
 736528696E7075745F737472293B6173736572742873746F493634436865636B28696E
 7075742E706172616D732E737570706C79293D3D3D747275652626747970656F662069
 6E7075742E706172616D732E6E616D653D3D3D27737472696E67272626747970656F66
 20696E7075742E706172616D732E73796D626F6C3D3D3D27737472696E672726267479
 70656F6620696E7075742E706172616D732E646563696D616C733D3D3D276E756D6265
 72272C274172677320636865636B206661696C65642E27293B676C6F62616C41747472
 69627574652E6374703D27312E30273B676C6F62616C4174747269627574652E6E616D
 653D696E7075742E706172616D732E6E616D653B676C6F62616C417474726962757465
 2E73796D626F6C3D696E7075742E706172616D732E73796D626F6C3B676C6F62616C41
 74747269627574652E646563696D616C733D696E7075742E706172616D732E64656369
 6D616C733B676C6F62616C4174747269627574652E746F74616C537570706C793D696E
 7436344D756C28696E7075742E706172616D732E737570706C792C706F7765724F6642
 617365313028676C6F62616C4174747269627574652E646563696D616C7329293B676C
 6F62616C4174747269627574652E636F6E74726163744F776E65723D73656E6465723B
 676C6F62616C4174747269627574652E62616C616E63653D676C6F62616C4174747269
 627574652E746F74616C537570706C793B73746F7261676553746F726528676C6F6261
 6C4174747269627574654B657928292C4A534F4E2E737472696E6769667928676C6F62
 616C41747472696275746529293B7D66756E6374696F6E206D61696E28696E7075745F
 737472297B6C657420696E7075743D4A534F4E2E706172736528696E7075745F737472
 293B696628696E7075742E6D6574686F643D3D3D277472616E7366657227297B747261
 6E7366657228696E7075742E706172616D732E746F2C696E7075742E706172616D732E
 76616C7565293B7D656C736520696628696E7075742E6D6574686F643D3D3D27747261
 6E7366657246726F6D27297B7472616E7366657246726F6D28696E7075742E70617261
 6D732E66726F6D2C696E7075742E706172616D732E746F2C696E7075742E706172616D
 732E76616C7565293B7D656C736520696628696E7075742E6D6574686F643D3D3D2761
 7070726F766527297B617070726F766528696E7075742E706172616D732E7370656E64
 65722C696E7075742E706172616D732E76616C7565293B7D656C736520696628696E70
 75742E6D6574686F643D3D3D2761737369676E27297B61737369676E28696E7075742E
 706172616D732E746F2C696E7075742E706172616D732E76616C7565293B7D656C7365
 20696628696E7075742E6D6574686F643D3D3D276368616E67654F776E657227297B63
 68616E67654F776E657228696E7075742E706172616D732E61646472657373293B7D65
 6C73657B7468726F77273C756E6964656E746966696564206F7065726174696F6E2074
 7970653E273B7D7D66756E6374696F6E20717565727928696E7075745F737472297B6C
 6F6164476C6F62616C41747472696275746528293B6C657420726573756C743D7B7D3B
 6C657420696E7075743D4A534F4E2E706172736528696E7075745F737472293B696628
 696E7075742E6D6574686F643D3D3D276E616D6527297B726573756C742E6E616D653D
 6E616D6528293B7D656C736520696628696E7075742E6D6574686F643D3D3D2773796D
 626F6C27297B726573756C742E73796D626F6C3D73796D626F6C28293B7D656C736520
 696628696E7075742E6D6574686F643D3D3D27646563696D616C7327297B726573756C
 742E646563696D616C733D646563696D616C7328293B7D656C736520696628696E7075
 742E6D6574686F643D3D3D27746F74616C537570706C7927297B726573756C742E746F
 74616C537570706C793D746F74616C537570706C7928293B7D656C736520696628696E
 7075742E6D6574686F643D3D3D2763747027297B726573756C742E6374703D63747028
 293B7D656C736520696628696E7075742E6D6574686F643D3D3D27636F6E7472616374
 496E666F27297B726573756C742E636F6E7472616374496E666F3D636F6E7472616374
 496E666F28293B7D656C736520696628696E7075742E6D6574686F643D3D3D2762616C
 616E63654F6627297B726573756C742E62616C616E63653D62616C616E63654F662869
 6E7075742E706172616D732E61646472657373293B7D656C736520696628696E707574
 2E6D6574686F643D3D3D27616C6C6F77616E636527297B726573756C742E616C6C6F77
 616E63653D616C6C6F77616E636528696E7075742E706172616D732E6F776E65722C69
 6E7075742E706172616D732E7370656E646572293B7D656C73657B7468726F77273C75
 6E6964656E746966696564206F7065726174696F6E20747970653E273B7D6C6F672872
 6573756C74293B72657475726E204A534F4E2E737472696E6769667928726573756C74
 293B7D1A041A02080128C0843D32577B22706172616D73223A7B2273796D626F6C223A
 2243474F222C22646563696D616C73223A382C226E616D65223A22436F6E7472616374
 20476C6F62616C222C22737570706C79223A2231303030303030303030227D7D




.. _签名交易-1:

签名交易
~~~~~~~~

所有的交易都需要对其进行签名，签名完成后交易才能生效。签名结果包括签名数据和公钥。
签名交易的具体代码如下，示例中的参数 transactionBlob 是调用 seralizeTransaction 得到的序列化交易字符串。

.. code:: javascript

 public Signature[] signTransaction(String transactionBlob) { 
    Signature[] signatures = null; 
    // The account private key to create contract and issue ctp 1.0 token 
 String senderPrivateKey = "privbs4iBCugQeb2eiycU8RzqkPqd28eaAYrRJGwtJTG8FVHjwAyjiyC"; 
 
 // Sign transaction BLob 
 TransactionSignRequest transactionSignRequest = new TransactionSignRequest(); 
 transactionSignRequest.setBlob(transactionBlob); 
 transactionSignRequest.addPrivateKey(senderPrivateKey); 
 TransactionSignResponse transactionSignResponse = sdk.getTransactionService().sign(transactionSignRequest); 
 if (transactionSignResponse.getErrorCode() == 0) { 
    signatures = transactionSignResponse.getResult().getSignatures(); 
 } else { 
    System.out.println("error: " + transactionSignResponse.getErrorDesc()); 
 } 
 return signatures; 
 } 

签名交易的返回值如下：

::

 signData: D6DBD26FA9E2B179209DD96F359491CE46B84C4E9EE3E85D646B1F67750D8D0DA2B9B51C9C22F165A3F3F4B16B52541C08C9AD266EE1E1CC86DC86D25E52290D 
 publicKey: b00135e99d67a4c2e10527f766e08bc6afd4420951628149042fdad6584a5321c23c716a528b 



.. _发送交易-1:


发送交易
~~~~~~~~~

发送交易即将序列化的交易和签名发送到 BuChain。
发送交易具体代码如下，示例中的参数 transactionBlob 是调用 seralizeTransaction 得到的序列化交易字符串，signatures 是调用 signTransaction 得到的签名数据。


.. code:: javascript

 public String submitTransaction(String transactionBlob, Signature[] signatures) { 
 String  hash = null; 
 
 // Submit transaction 
 TransactionSubmitRequest transactionSubmitRequest = new TransactionSubmitRequest(); 
 transactionSubmitRequest.setTransactionBlob(transactionBlob); 
 transactionSubmitRequest.setSignatures(signatures); 
 TransactionSubmitResponse transactionSubmitResponse = sdk.getTransactionService().submit(transactionSubmitRequest); 
 if (0 == transactionSubmitResponse.getErrorCode()) { 
        hash = transactionSubmitResponse.getResult().getHash(); 
 } else { 
        System.out.println("error: " + transactionSubmitResponse.getErrorDesc()); 
 } 
 return  hash ; 
 } 

发送交易的返回值如下：

::
 
 hash: 514d8caf81a78429622794ea8e5ebe8b1c7dd4b7e56c668eb890aa3a35c239ab



.. _查询交易是否执行成功-1:


查询交易是否执行成功
~~~~~~~~~~~~~~~~~~

.. note:: 发送交易返回的结果只是交易是否提交成功的结果，而交易是否执行成功的结果需要执行如下操作进行查询, 具体有两种方法：


区块链浏览器查询
^^^^^^^^^^^^^^^

在BUMO区块链浏览器中查询上面的hash，主网(https://explorer.bumo.io)，测试网(http://explorer.bumotest.io)，操作如下图：

|BUExplorer1|

查询结果如下：


|BUResult1|


调用接口查询
^^^^^^^^^^^^

调用接口查询的代码如下，示例中的参数 txHash 是调用 submitTransaction 得到的交易哈希（交易的惟一标识）。

::

 public boolean checkTransactionStatus(String txHash) {
    Boolean transactionStatus = false;

 // 交易执行等待10秒
 try {
    Thread.sleep(10000);
 } catch (InterruptedException e) {
    e.printStackTrace();
 }
 // Init request
 TransactionGetInfoRequest request = new TransactionGetInfoRequest();
 request.setHash(txHash);

 // Call getInfo
 TransactionGetInfoResponse response = sdk.getTransactionService().getInfo(request);
 if (response.getErrorCode() == 0) {
    transactionStatus = true;
 } else {
    System.out.println("error: " + response.getErrorDesc());
  }
 return transactionStatus;
 }


返回结果如下：

::
 
 transactionStatus: true


实例场景二
^^^^^^^^^^

资方 ``buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH`` 在 BuChain 上通过智能合约账户 ``buQcEk2dpUv6uoXjAqisVRyP1bBSeWUHCtF2`` 分配给自己 20000 CGO，
并将 10000 CGO转移给另一个账户 ``buQXPeTjT173kagZ7j8NWAPJAgJCpJHFdyc7`` 。

线上demo请看: `TriggerContractDemo.java <https://github.com/bumoproject/bumo-sdk-java/blob/develop/examples/src/main/java/io/bumo/sdk/example/TriggerContractDemo.java>`_。


本场景的具体执行过程包括 :ref:`创建SDK实例-2`、:ref:`获取资方账户的序列号-2`、`组装分配CGO和转移CGO`_、:ref:`序列化交易-2`、:ref:`签名交易-2`、:ref:`发送交易-2`、:ref:`查询交易是否执行成功-2`。






.. _创建SDK实例-2:

创建SDK实例
~~~~~~~~~~~

创建实例并设置url(部署的某节点的IP和端口)。

::

 String url = "http://seed1.bumotest.io:26002";
 SDK sdk = SDK.getInstance(url);

在 BuChain 网络里，每个区块产生时间是10秒，每个交易只需要一次确认即可得到交易终态。

环境说明如下：

+-------------------------+--------------------+------------------+----------------------------------+
| 网络环境                | IP                 | Port             | 区块链浏览器                     |
+=========================+====================+==================+==================================+
| 主网                    | seed1.bumo.io      | 16002            | https://explorer.bumo.io         |
+-------------------------+--------------------+------------------+----------------------------------+
| 测试                    | seed1.bumotest.io  | 26002            | http://explorer.bumotest.io      |
+-------------------------+--------------------+------------------+----------------------------------+


.. _获取资方账户的序列号-2:

获取资方账户的序列号
~~~~~~~~~~~~~~~~~~~

每个账户都维护着自己的序列号，该序列号从1开始，依次递增，一个序列号标志着一个该账户的交易。
获取资方账号序列号的代码如下：

::

 public long getAccountNonce() {
 long nonce = 0;

    // Init request
    String accountAddress = [资方账户地址];
    AccountGetNonceRequest request = new AccountGetNonceRequest();
    request.setAddress(accountAddress);

    // Call getNonce
    AccountGetNonceResponse response = sdk.getAccountService().getNonce(request);
    if (0 == response.getErrorCode()) {
        nonce = response.getResult().getNonce();
    } else {
        System.out.println("error: " + response.getErrorDesc());
 }
 return nonce;
 }

返回值如下：

::

 nonce: 2







组装分配CGO和转移CGO
~~~~~~~~~~~~~~~~~~~~

本章节包含两个操作：分配CGO和转移CGO。以下为示例代码：

.. code:: javascript

 
 public BaseOperation[] buildOperations() 
 { // The account address to issue apt1.0 token 
 String invokeAddress = "buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH"; 
 // The contract address 
 String contractAddress = "buQcEk2dpUv6uoXjAqisVRyP1bBSeWUHCtF2"; 
 // The destination address 
 String destAddress = "buQXPeTjT173kagZ7j8NWAPJAgJCpJHFdyc7"; 
 // The amount to be assigned 
 String assignAmount = "20000"; 
 // The amount to be transfered 
 String transferAmount = "10000";


 // build assign method input 
 JSONObject assignInput = new JSONObject(); 
 assignInput.put("method", "assign"); 
 JSONObject assignParams = new JSONObject(); 
 assignParams.put("to", invokeAddress); 
 assignParams.put("value", assignAmount); 
 assignInput.put("params", assignParams); 

 // build send bu operation to assign CGO 
 ContractInvokeByBUOperation assignOperation = new ContractInvokeByBUOperation(); 
 assignOperation.setSourceAddress(invokeAddress); 
 assignOperation.setContractAddress(contractAddress); 
 assignOperation.setBuAmount(0L); 
 assignOperation.setInput(assignInput.toJSONString());

 // build transfer method input 
 JSONObject transferInput = new JSONObject(); 
 transferInput.put("method", "transfer"); 
 JSONObject transferParams = new JSONObject(); 
 transferParams.put("to", destAddress); 
 transferParams.put("value", transferAmount); 
 transferInput.put("params", transferParams);

 // build send bu operation to transfer CGO 
 ContractInvokeByBUOperation transferOperation = new ContractInvokeByBUOperation(); 
 transferOperation.setSourceAddress(invokeAddress); 
 transferOperation.setContractAddress(contractAddress); 
 transferOperation.setBuAmount(0L); 
 transferOperation.setInput(transferInput.toJSONString()); 
 BaseOperation[] operations = { assignOperation, transferOperation }; 
 return operations; }























.. _序列化交易-2:

序列化交易
~~~~~~~~~~

序列化交易以便网络传输。


.. note:: - feeLimit: 本次交易发起方最多支付本次交易的交易费用，创建合约账户并发行ctp token操作请填写0.02 BU。

       - nonce: 本次交易发起方的交易序列号，该值由当前账户的 nonce 值加1得到。 



序列化交易的具体代码如下，示例中的参数 nonce 是调用 getAccountNonce 得到的账户序列号，参数 operations 是调用 buildOperations 得到发行资产的操作。
以下是序列化交易的示例代码：

.. code:: JavaScript

 public String seralizeTransaction(Long nonce,  BaseOperation[] operations) { 
 String transactionBlob = null; 
 
 // The account address to create contract and issue ctp 1.0 token 
 String senderAddresss = "buQYLtRq4j3eqbjVNGYkKYo3sLBqW3TQH2xH"; 
    // The gasPrice is fixed at 1000L, the unit is MO 
    Long gasPrice = 1000L; 
    // Set up the maximum cost 10.08BU 
    Long feeLimit = ToBaseUnit.BU2MO("0.02"); 
    // Nonce should add 1 
 nonce += 1; 
 
 // Build transaction  Blob 
 TransactionBuildBlobRequest transactionBuildBlobRequest = new TransactionBuildBlobRequest(); 
 transactionBuildBlobRequest.setSourceAddress(senderAddresss); 
 transactionBuildBlobRequest.setNonce(nonce); 
 transactionBuildBlobRequest.setFeeLimit(feeLimit); 
 transactionBuildBlobRequest.setGasPrice(gasPrice); 
 for (int i = 0; i < operations.length; i++) { 
    transactionBuildBlobRequest.addOperation(operations[i]); 
 } 
 TransactionBuildBlobResponse transactionBuildBlobResponse = sdk.getTransactionService().buildBlob(transactionBuildBlobRequest); 
 if (transactionBuildBlobResponse.getErrorCode() == 0) { 
 transactionBlob = transactionBuildBlobResponse. getResult().getTransactionBlob(); 
 } else { 
    System.out.println("error: " + transactionBuildBlobResponse.getErrorDesc()); 
 } 
 return transactionBlob; 
 } 

返回值为：

::

 transactionBlob: 
 0A24627551594C745271346A336571626A564E47596B4B596F33734C4271573354514832784810031
 880B8D3E00320E8073AAD0108071224627551594C74527346A336571626A564E47596B4B596F33734
 C427157335451483278485282010A2462755163456B326470557636756F586A417169735652795031
 62425365575548437446321A5A7B226D6574686F64223A2261737369676E222C22706172616D73223
 A7B22746F223A22627551594C745271346A336571626A564E47596B4B596F33734C42715733545148
 327848222C2276616C7565223A223230303030227D7D3AAF0108071224627551594C745271346A336
 571626A564E47596B4B596F33734C427157335451483278485284010A2462755163456B3264705576
 36756F586A41716973565279503162425365575548437446321A5C7B226D6574686F64223A2274726
 16E73666572222C22706172616D73223A7B22746F223A22627551585065546A543137336B61675A37
 6A384E5741504A41674A43704A484664796337222C2276616C7565223A223130303030227D7D 














.. _签名交易-2:

签名交易
~~~~~~~~

所有的交易都需要经过签名后，才是有效的。签名结果包括签名数据和公钥。
以下是签名交易的示例代码，示例中的参数 transactionBlob 是调用 seralizeTransaction 得到的序列化交易字符串。

.. code:: JavaScript

 public Signature[] signTransaction(String transactionBlob) { 
    Signature[] signatures = null; 
    // The account private key to create contract and issue ctp 1.0 token 
 String senderPrivateKey = "privbs4iBCugQeb2eiycU8RzqkPqd28eaAYrRJGwtJTG8FVHjwAyjiyC"; 
 
 // Sign transaction BLob 
 TransactionSignRequest transactionSignRequest = new TransactionSignRequest(); 
 transactionSignRequest.setBlob(transactionBlob); 
 transactionSignRequest.addPrivateKey(senderPrivateKey); 
 TransactionSignResponse transactionSignResponse = sdk.getTransactionService().sign(transactionSignRequest); 
 if (transactionSignResponse.getErrorCode() == 0) { 
    signatures = transactionSignResponse.getResult().getSignatures(); 
 } else { 
    System.out.println("error: " + transactionSignResponse.getErrorDesc()); 
 } 
 return signatures; 
 } 
 
返回值为：

::

 signData: F13B762108993206BABC785BB49DF2353411E3ED4E5996BA2E8E01EB0E64AB48DA57074D841C34CE4D3E494EA0643D9C683529732989322EFCE448A06B5C1900 
 publicKey: b00135e99d67a4c2e10527f766e08bc6afd4420951628149042fdad6584a5321c23c716a528b 





.. _发送交易-2:

发送交易
~~~~~~~~

发送交易即将序列化的交易和签名发送到 BuChain。
以下为发送交易的代码示例，示例中的参数 transactionBlob 是调用 seralizeTransaction 得到的序列化交易字符串，signatures 是调用 signTransaction 得到的签名数据。

.. code:: JavaScript

 public String submitTransaction(String transactionBlob, Signature[] signatures) { 
 String  hash = null; 
 
 // Submit transaction 
 TransactionSubmitRequest transactionSubmitRequest = new TransactionSubmitRequest(); 
 transactionSubmitRequest.setTransactionBlob(transactionBlob); 
 transactionSubmitRequest.setSignatures(signatures); 
 TransactionSubmitResponse transactionSubmitResponse = sdk.getTransactionService().submit(transactionSubmitRequest); 
 if (0 == transactionSubmitResponse.getErrorCode()) { 
        hash = transactionSubmitResponse.getResult().getHash(); 
 } else { 
        System.out.println("error: " + transactionSubmitResponse.getErrorDesc()); 
 } 
 return  hash ; 
 } 

返回值为：

::

 hash: 6434743a136c0d03d41bb48146c65ebefc7014154b4160f3b9d3b9c50eb47054


.. _查询交易是否执行成功-2:

查询交易是否执行成功
~~~~~~~~~~~~~~~~~~~~

.. note:: 发送交易返回的结果只是交易是否提交成功的结果，而交易是否执行成功的结果需要执行如下查询操作, 具体有两种方法：


区块链浏览器查询
^^^^^^^^^^^^^^^

在BUMO区块链浏览器中查询上面的hash，主网(https://explorer.bumo.io)，测试网(http://explorer.bumotest.io)，操作如下图：

|BUExplorer2|

查询结果如下：


|BUResult2|


调用接口查询
^^^^^^^^^^^^

调用接口查询的代码如下，示例中的参数 txHash 是调用 submitTransaction 得到的交易哈希（交易的惟一标识）。

.. code:: javascript

 public boolean checkTransactionStatus(String txHash) { 
    Boolean transactionStatus = false; 
    // 调用上面封装的“发送交易”接口 
 // 交易执行等待10秒 
 try { 
    Thread.sleep(10000); 
 } catch (InterruptedException e) { 
    e.printStackTrace(); 
 } 
 // Init request 
 TransactionGetInfoRequest request = new TransactionGetInfoRequest(); 
 request.setHash(txHash); 
 
 // Call getInfo 
 TransactionGetInfoResponse response = sdk.getTransactionService().getInfo(request); 
 if (response.getErrorCode() == 0) { 
    transactionStatus = true; 
 } else { 
    System.out.println("error: " + response.getErrorDesc()); 
 } 
 return transactionStatus; 
 } 






返回值为：

::
 
 transactionStatus: true


实例场景三
^^^^^^^^^^

在 BuChain 上通过智能合约账户 ``buQcEk2dpUv6uoXjAqisVRyP1bBSeWUHCtF2`` 查询账户 ``buQXPeTjT173kagZ7j8NWAPJAgJCpJHFdyc7`` 的 CGO 的余额。
本节主要讲解 :ref:`创建SDK实例-3` 和 `查询余额`_ 。


.. _创建SDK实例-3:

创建SDK实例
~~~~~~~~~~~~

创建实例并设置url(部署的某节点的IP和端口)。

::

 String url = "http://seed1.bumotest.io:26002";
 SDK sdk = SDK.getInstance(url);

在 BuChain 网络里，每个区块产生时间是10秒，每个交易只需要一次确认即可得到交易终态。
环境说明如下：

+-------------------------+--------------------+------------------+----------------------------------+
| 网络环境                | IP                 | Port             | 区块链浏览器                     |
+=========================+====================+==================+==================================+
| 主网                    | seed1.bumo.io      | 16002            | https://explorer.bumo.io         |
+-------------------------+--------------------+------------------+----------------------------------+
| 测试                    | seed1.bumotest.io  | 26002            | http://explorer.bumotest.io      |
+-------------------------+--------------------+------------------+----------------------------------+

查询余额
~~~~~~~~

查询余额的代码示例如下：

.. code:: JavaScript

 public String queryContract() { 
    // Init variable 
    // Contract address 
    String contractAddress = "buQcEk2dpUv6uoXjAqisVRyP1bBSeWUHCtF2"; 
    // TokenOwner address 
    String tokenOwner = "buQXPeTjT173kagZ7j8NWAPJAgJCpJHFdyc7"; 
 
    // Init input 
    JSONObject input = new JSONObject(); 
 input.put("method", "balanceOf"); 
 JSONObject params = new JSONObject(); 
 params.put("address", tokenOwner); 
 input.put("params", params); 
 // Init request 
    ContractCallRequest request = new ContractCallRequest(); 
 request.setContractAddress(contractAddress); 
 request.setFeeLimit(10000000000L); 
 request.setOptType(2); 
    request.setInput(input.toJSONString()); 
 
    // Call call 
    String result = null; 
    ContractCallResponse response = sdk.getContractService().call(request); 
    if (response.getErrorCode() == 0) { 
        result = JSON.toJSONString(response.getResult().getQueryRets().getJSONObject(0)); 
    } else { 
        System.out.println("error: " + response.getErrorDesc()); 
    } 
 return result; 
 } 

返回值为：

::

 result: {"result":{"type":"string","value":"{\"balance\":\"10000\"}"}} 

 




.. |warnings| image:: image/warnings.png
.. |nowarnings| image:: image/nowarnings.png
.. |compressedString| image:: image/compressedString.png
.. |BUExplorer1| image:: image/BUExplorer1.png
.. |BUExplorer2| image:: image/BUExplorer2.png
.. |BUResult1| image:: image/BUResult1.png
.. |BUResult2| image:: image/BUResult2.png

















































































