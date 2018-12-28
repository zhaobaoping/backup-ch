BUMO GO SDK 指南 
================

概述
----

本文档对BUMO GO SDK常用的接口进行了详细说明,
使开发者能够更方便地写入和查询BU区块链。

包导入
----

项目所依赖的包在src文件夹中，获取包的方法如下：

::

 //获取包
 go get github.com/bumoproject/bumo-sdk-go

术语
----

本章节对该文档中使用到的术语进行了详细说明。

**操作BU区块链** 

操作BU区块链是指向BU区块链写入或修改数据。

**提交交易**

提交交易是指向BU区块链发送修改的内容。

**查询BU区块链**

查询BU区块链是指查询BU区块链中的数据。

**账户服务**

账户服务是指提供账户相关的有效性校验与查询接口。

**资产服务**

资产服务是指提供资产相关的查询接口。

**Ctp10Token服务**

Ctp10Token服务是指提供合约资产相关的有效性校验与查询接口。

**合约服务**

合约服务是指提供合约相关的有效性校验与查询接口。

**交易服务**

交易服务是指提供交易相关的提交与查询接口。

**区块服务**

区块服务是指提供区块的查询接口。

**账户nonce值**

账户nonce值是指每个账户都维护的一个序列号，用于用户提交交易时标识交易的执行顺序。

请求参数与响应数据格式
--------------------

本章节将详细介绍请求参数与响应数据的格式。

请求参数
~~~~~~~~

接口的请求参数的类名，由 **类名+方法名+Request** 构成，例如:
账户服务下的 ``getInfo`` 接口的请求参数格式是 ``AccountGetInfoRequest`` 。

请求参数的成员，是各个接口的入参成员。例如：账户服务下的 ``getInfo`` 接口的入参成员是 ``address`` ，那么该接口的请求参数的完整结构如下：

::

   type AccountGetInfoRequest struct {
   address string
   }

响应数据
~~~~~~~~

响应数据包含错误码，错误描述和result，响应数据的类名由 **类名+方法名+Response** 构成。

例如: ``Account.GetInfo()`` 的结构体名是 ``AccountGetInfoResponse`` ：

::

 type AccountGetInfoResponse struct {
   ErrorCode int
   ErrorDesc string
   Result  AccountGetInfoResult
 }

.. note:: - ErrorCode: 0表示无错误，大于0表示有错误。

       - ErrorDesc: 空表示无错误，有内容表示有错误。

       - Result: 返回结果的结构体，其中结构体由 **类名+方法名+Result** 构成。例如：``Account.GetNonce()`` 的结构体名是 ``AccountGetNonceResult`` 。 
        
::

    type AccountGetNonceResult struct {
      Nonce int64
    }

使用方法
--------

这里介绍SDK的使用流程，首先需要生成SDK实例，然后调用相应服务的接口，其中服务包括账户服务、资产服务、合约服务、交易服务、区块服务，接口按使用分类分为生成公私钥地址接口、有效性校验接口、查询接口、提交交易相关接口。

包导入
~~~~~~

生成SDK实例之前导入使用的包：

::

 import(
   "github.com/bumoproject/bumo-sdk-go/src/model"
   "github.com/bumoproject/bumo-sdk-go/src/sdk"
 )

生成SDK实例
~~~~~~~~~~~

初始化SDK结构方法：

::

 var testSdk sdk.sdk

调用SDK的接口Init：

::

 url :="http://seed1.bumotest.io:26002"
 var reqData model.SDKInitRequest
 reqData.SetUrl(url)
 resData := testSdk.Init(reqData)

生成公私钥地址
~~~~~~~~~~~~~

通过调用Account的Create生成账户，方法如下：

::

 resData :=testSdk.Account.Create()

有效性校验
~~~~~~~~~~

有效性校验接口用于校验信息的有效性，直接调用相应的接口即可，比如，校验账户地址有效性，调用如下：

::

 //初始化传入参数
 var reqData model.AccountCheckValidRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 //调用接口检查
 resData := testSdk.Account.CheckValid(reqData)

查询
~~~~

使用查询接口时可直接调用，如查询账户信息方法如下：

::

 //初始化传入参数
 var reqData model.AccountGetInfoRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 //调用接口查询 
 resData := testSdk.Account.GetInfo(reqData)

提交交易
~~~~~~~~

提交交易的过程包括以下几步：

`1. 获取账户nonce值`_

`2. 构建操作`_

`3. 构建交易Blob`_

`4. 签名交易`_

`5. 广播交易`_


1. 获取账户nonce值
^^^^^^^^^^^^^^^^^^

开发者可自己维护各个账户nonce，在提交完一个交易后，nonce值自动递增1，这样可以在短时间内发送多笔交易；否则，必须等上一个交易执行完成后，账户的nonce值才会加1。接口调用如下：

::

 //初始化请求参数
 var reqData model.AccountGetNonceRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 //调用GetNonce接口
 resData := testSdk.Account.GetNonce(reqData)

2. 构建操作
^^^^^^^^^^^

这里的操作是指在交易中做的一些动作。例如：构建发送BU操作BUSendOperation，调用如下:

::

 var buSendOperation model.BUSendOperation
 buSendOperation.Init()
 var amount int64 = 100
 var address string = "buQVU86Jm4FeRW4JcQTD9Rx9NkUkHikYGp6z"
 buSendOperation.SetAmount(amount)
 buSendOperation.SetDestAddress(address)

3. 构建交易Blob
^^^^^^^^^^^^^^^

构建交易Blob接口用于生成交易Blob串，接口调用如下：

::

 //初始化传入参数
 var reqDataBlob model.TransactionBuildBlobRequest
 reqDataBlob.SetSourceAddress(sourceAddress)
 reqDataBlob.SetFeeLimit(feeLimit)
 reqDataBlob.SetGasPrice(gasPrice)
 reqDataBlob.SetNonce(senderNonce)
 reqDataBlob.SetOperation(buSendOperation)
 //调用BuildBlob接口
 resDataBlob := testSdk.Transaction.BuildBlob(reqDataBlob)

.. note:: gasPrice和feeLimit的单位是MO，且 1 BU =10^8 MO。

4. 签名交易
^^^^^^^^^^^

签名交易接口用于交易发起者使用私钥对交易进行签名。接口调用如下：

::

 //初始化传入参数
 PrivateKey := []string{"privbUPxs6QGkJaNdgWS2hisny6ytx1g833cD7V9C3YET9mJ25wdcq6h"}
 var reqData model.TransactionSignRequest
 reqData.SetBlob(resDataBlob.Result.Blob)
 reqData.SetPrivateKeys(PrivateKey)
 //调用Sign接口
 resDataSign := testSdk.Transaction.Sign(reqData)

5. 广播交易
^^^^^^^^^^^

广播交易接口用于向BU区块链发送交易，触发交易的执行。接口调用如下：

::

 //初始化传入参数
 var reqData model.TransactionSubmitRequest
 reqData.SetBlob(resDataBlob.Result.Blob)
 reqData.SetSignatures(resDataSign.Result.Signatures)
 //调用Submit接口
 resDataSubmit := testSdk.Transaction.Submit(reqData)

账户服务
--------

账户服务主要是账户相关的接口，包括7个接口： ``CheckValid``、``Create``、``GetInfo-Account``、``GetNonce``、
``GetBalance-Account``、``GetAssets``、``GetMetadata``。

CheckValid
~~~~~~~~~~

``CheckValid`` 接口用于检测账户地址的有效性。

调用方法如下：

::

 CheckValid(model.AccountCheckValidRequest)model.AccountCheckValidResponse

请求参数如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| address | string | 待检测的账户地址 |
+---------+--------+------------------+

响应数据如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| IsValid | string | 账户地址是否有效 |
+---------+--------+------------------+

错误码如下表：

+--------------+--------+--------------+
| 错误信息     | 错误码 | 描述         |
+==============+========+==============+
| SYSTEM_ERROR | 20000  | System error |
+--------------+--------+--------------+

具体示例如下所示：

::

   var reqData model.AccountCheckValidRequest
   address := "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
   reqData.SetAddress(address)
   resData := testSdk.Account.CheckValid(reqData)
   if resData.ErrorCode == 0 {
     fmt.Println(resData.Result.IsValid)
   }

Create
~~~~~~

``Create`` 接口用于形成私钥对。

调用方法如下：

::

 Create() model.AccountCreateResponse

响应数据如下表：

+------------+--------+------+
| 参数       | 类型   | 描述 |
+============+========+======+
| PrivateKey | string | 私钥 |
+------------+--------+------+
| PublicKey  | string | 公钥 |
+------------+--------+------+
| Address    | string | 地址 |
+------------+--------+------+

具体示例如下所示：

::

 resData := testSdk.Account.Create()
 if resData.ErrorCode == 0 {
   fmt.Println("Address:",resData.Result.Address)
   fmt.Println("PrivateKey:",resData.Result.PrivateKey)
   fmt.Println("PublicKey:",resData.Result.PublicKey)
 }

GetInfo-Account
~~~~~~~~~~~~~~~

``GetInfo-Account`` 接口用于查询账户信息。

调用方法如下：

::

 GetInfo(model.AccountGetInfoRequest) model.AccountGetInfoResponse

请求参数如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| address | string | 待检测的账户地址 |
+---------+--------+------------------+

响应数据如下表：

+---------+------------------+----------------+
| 参数    | 类型             | 描述           |
+=========+==================+================+
| Address | string           | 账户地址       |
+---------+------------------+----------------+
| Balance | int64            | 账户余额       |
+---------+------------------+----------------+
| Nonce   | int64            | 账户交易序列号 |
+---------+------------------+----------------+
| Priv    | `Priv`_          | 账户权限       |
+---------+------------------+----------------+ 


错误码如下表：

+-----------------------+--------+-------------------------+
| 错误信息              | 错误码 | 描述                    |
+=======================+========+=========================+
| INVALID_ADDRESS_ERROR | 11006  | Invalid address         |
+-----------------------+--------+-------------------------+
| CONNECTNETWORK_ERROR  | 11007  | Failed to connect to    |
|                       |        | the blockchain          |
+-----------------------+--------+-------------------------+
| SYSTEM_ERROR          | 20000  | System error            |
+-----------------------+--------+-------------------------+

具体示例如下所示：

::

 var reqData model.AccountGetInfoRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 resData := testSdk.Account.GetInfo(reqData)
 if resData.ErrorCode == 0 {
   data, _ := json.Marshal(resData.Result)
   fmt.Println("Info:", string(data))
 }

接口对象类型参考
^^^^^^^^^^^^^^^

下面是 ``GetInfo-Account`` 的接口对象类型：

Priv
++++

+--------------+----------------+--------------+
| 参数         | 类型           | 描述         |
+==============+================+==============+
| MasterWeight | int64          | 账户自身权重 |
+--------------+----------------+--------------+
| Signers      | [] `Signer`_   | 签名者权重   |
+--------------+----------------+--------------+
| Thresholds   | `Threshold`_   | 门限         |
+--------------+----------------+--------------+


Signer
++++++

+---------+--------+--------------+
| 参数    | 类型   | 描述         |
+=========+========+==============+
| Address | string | 签名账户地址 |
+---------+--------+--------------+
| Weight  | int64  | 签名账户权重 |
+---------+--------+--------------+

Threshold
+++++++++

+----------------+-------------------+--------------------+
| 参数           | 类型              | 描述               |
+================+===================+====================+
| TxThreshold    | string            | 交易默认门限       |
+----------------+-------------------+--------------------+
| TypeThresholds | `TypeThreshold`_  | 不同类型交易的门限 |
+----------------+-------------------+--------------------+

TypeThreshold
++++++++++++++

+-----------+-------+----------+
| 参数      | 类型  | 描述     |
+===========+=======+==========+
| Type      | int64 | 操作类型 |
+-----------+-------+----------+
| Threshold | int64 | 门限     |
+-----------+-------+----------+

GetNonce
~~~~~~~~

``GetNonce`` 接口用于获取账户的nonce值。

调用方法如下：

::

 GetNonce(model.AccountGetNonceRequest)model.AccountGetNonceResponse

请求参数如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| address | string | 待检测的账户地址 |
+---------+--------+------------------+

响应数据如下表：

+---------+--------+--------------------+
| 参数    | 类型   | 描述               |
+=========+========+====================+
| nonce   | long   | 该账户的交易序列号 |
+---------+--------+--------------------+

错误码如下表：

+-----------------------+--------+-------------------------+
| 错误信息              | 错误码 | 描述                    |
+=======================+========+=========================+
| INVALID_ADDRESS_ERROR | 11006  | Invalid address         |
+-----------------------+--------+-------------------------+
| CONNECTNETWORK_ERROR  | 11007  | Failed to connect to    |
|                       |        | the network             |
+-----------------------+--------+-------------------------+
| SYSTEM_ERROR          | 20000  | System error            |
+-----------------------+--------+-------------------------+

具体示例如下所示：

::

 var reqData model.AccountGetNonceRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 if resData.ErrorCode == 0 {
   fmt.Println(resData.Result.Nonce)
 }

GetBalance-Account
~~~~~~~~~~~~~~~~~~~

``GetBalance-Account`` 接口用于获取账户的Balance值。

调用方法如下：

::

 GetBalance(model.AccountGetBalanceRequest)model.AccountGetBalanceResponse

请求参数如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| address | string | 待检测的账户地址 |
+---------+--------+------------------+

响应数据如下表：

+---------+-------+--------------+
| 参数    | 类型  | 描述         |
+=========+=======+==============+
| Balance | int64 | 该账户的余额 |
+---------+-------+--------------+

错误码如下表：

+-----------------------+--------+-------------------------+
| 错误信息              | 错误码 | 描述                    |
+=======================+========+=========================+
| INVALID_ADDRESS_ERROR | 11006  | Invalid address         |
+-----------------------+--------+-------------------------+
| CONNECTNETWORK_ERROR  | 11007  | Failed to connect to    |
|                       |        | the network             |
+-----------------------+--------+-------------------------+
| SYSTEM_ERROR          | 20000  | System error            |
+-----------------------+--------+-------------------------+

具体示例如下所示：

::

 var reqData model.AccountGetBalanceRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 resData := testSdk.Account.GetBalance(reqData)
 if resData.ErrorCode == 0 {
   fmt.Println("Balance", resData.Result.Balance)
 }

GetAssets
~~~~~~~~~~

``GetAssets`` 接口用于获取账户的Asset值。

调用方法如下：

::

 GetAssets(model.AccountGetAssetsRequest)model.AccountGetAssetsResponse

请求参数如下表：

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| address | string | 待检测的账户地址 |
+---------+--------+------------------+

响应数据如下表：

+--------+--------------+----------+
| 参数   | 类型         | 描述     |
+========+==============+==========+
| Assets | [] `Asset`_  | 账户资产 |
+--------+--------------+----------+

错误码如下表：

+-----------------------+--------+-------------------------+
| 错误信息              | 错误码 | 描述                    |
+=======================+========+=========================+
| INVALID_ADDRESS_ERROR | 11006  | Invalid address         |
+-----------------------+--------+-------------------------+
| CONNECTNETWORK_ERROR  | 11007  | Failed to connect to    |
|                       |        | the network             |
+-----------------------+--------+-------------------------+
| SYSTEM_ERROR          | 20000  | System error            |
+-----------------------+--------+-------------------------+

具体示例如下所示：

::

 var reqData model.AccountGetAssetsRequest
 var address string = "buQtfFxpQP9JCFgmu4WBojBbEnVyQGaJDgGn"
 reqData.SetAddress(address)
 resData := testSdk.Account.GetAssets(reqData)
 if resData.ErrorCode == 0 {
   data, _ := json.Marshal(resData.Result.Assets)
   fmt.Println("Assets:", string(data))
 }

接口对象类型参考
^^^^^^^^^^^^

下面是 ``GetAssets`` 的接口对象类型：

Asset
+++++

+--------+---------+--------------+
| 参数   | 类型    | 描述         |
+========+=========+==============+
| Key    | `key`_  | 资产惟一标识 |
+--------+---------+--------------+
| Amount | int64   | 资产数量     |
+--------+---------+--------------+

Key
++++

+--------+--------+----------------------+
| 参数   | 类型   | 描述                 |
+========+========+======================+
| Code   | string | 资产编码，长度[1 64] |
+--------+--------+----------------------+
| Issuer | string | 资产发行账户地址     |
+--------+--------+----------------------+

GetMetadata
~~~~~~~~~~~~

``GetMetadata`` 接口用来获取账户的Metadata信息。

调用方法如下：

::

 GetMetadata(model.AccountGetMetadataRequest)model.AccountGetMetadataResponse

请求参数如下表：

+---------+--------+-------------------------------------+
| 参数    | 类型   | 描述                                |
+=========+========+=====================================+
| address | string | 待检测的账户地址                    |
+---------+--------+-------------------------------------+
| key     | string | 选填，metadata关键字，长度[1, 1024] |
+---------+--------+-------------------------------------+

响应数据如下表：

+-----------+-----------------------+------+
| 参数      | 类型                  | 描述 |
+===========+=======================+======+
| Metadatas | [] :ref:`Metadata-1`  | 账户 |
+-----------+-----------------------+------+


错误码如下表：

+-----------------------+--------+----------------------------------------------+
| 错误信息              | 错误码 | 描述                                         |
+=======================+========+==============================================+
| INVALID_ADDRESS_ERROR | 11006  | Invalid address                              |
+-----------------------+--------+----------------------------------------------+
| CONNECTNETWORK_ERROR  | 11007  | Failed to connect to the network             |
+-----------------------+--------+----------------------------------------------+
| INVALID_DATAKEY_ERROR | 11011  | The length of key must be between 1 and 1024 |
+-----------------------+--------+----------------------------------------------+
| SYSTEM_ERROR          | 20000  | System error                                 |
+-----------------------+--------+----------------------------------------------+

具体示例如下所示：

::

 var reqData model.AccountGetMetadataRequest
 var address string = "buQemmMwmRQY1JkcU7w3nhruoX5N3j6C29uo"
 reqData.SetAddress(address)
 resData := testSdk.Account.GetMetadata(reqData)
 if resData.ErrorCode == 0 {
   data, _ := json.Marshal(resData.Result.Metadatas)
   fmt.Println("Metadatas:", string(data))
 }

接口对象类型参考
^^^^^^^^^^^^^^^

下面是 ``GetMetadata`` 的接口对象类型：

.. _Metadata-1:

Metadata
+++++++++

+---------+--------+------------------+
| 参数    | 类型   | 描述             |
+=========+========+==================+
| Key     | string | metadata的关键词 |
+---------+--------+------------------+
| Value   | string | metadata的内容   |
+---------+--------+------------------+
| Version | int64  | metadata的版本   |
+---------+--------+------------------+

资产服务
--------

资产服务主要是资产相关的接口，目前有1个接口：``GetInfo`` 。

GetInfo-Asset
~~~~~~~~~~~~~

``GetInfo-Asset`` 接口用于获取账户指定资产数量。

调用方法如下：

::

 GetInfo(model.AssetGetInfoRequest) model.AssetGetInfoResponse

请求参数如下表：

+---------+--------+-----------------------------+
| 参数    | 类型   | 描述                        |
+=========+========+=============================+
| address | string | 必填，待查询的账户地址      |
+---------+--------+-----------------------------+
| code    | string | 必填，资产编码，长度[1, 64] |
+---------+--------+-----------------------------+
| issuer  | string | 必填，资产发行账户地址      |
+---------+--------+-----------------------------+

响应数据如下表：

+--------+-----------------+----------+
| 参数   | 类型            | 描述     |
+========+=================+==========+
| Assets | [] `asset`_     | 账户资产 |
+--------+-----------------+----------+

错误码如下表：

+--------------------------+-----------+------------------+
| 错误信息                 | 错误码    | 描述             |
+==========================+===========+==================+
| INVALID_ADDRESS_ERROR    | 11006     | Invalid address  |
+--------------------------+-----------+------------------+
| CONNECTNETWORK_ERROR     | 11007     | Failed to connect|
|                          |           | to the network   |
+--------------------------+-----------+------------------+
| INVALID_ASSET_CODE_ERROR | 11023     | The length of    |
|                          |           | code must        |
|                          |           | be between 1 and |
|                          |           | 1024             |
+--------------------------+-----------+------------------+
| INVALID_ISSUER_ADDRESS   | 11027     | Invalid issuer   |
| _ERROR                   |           | address          |
+--------------------------+-----------+------------------+
| SYSTEM_ERROR             | 20000     | System error     |
+--------------------------+-----------+------------------+

具体示例如下所示：

::

 var reqData model.AssetGetInfoRequest
 var address string = "buQemmMwmRQY1JkcU7w3nhruoX5N3j6C29uo"
 reqData.SetAddress(address)
 reqData.SetIssuer("buQnc3AGCo6ycWJCce516MDbPHKjK7ywwkuo")
 reqData.SetCode("HNC")
 resData := testSdk.Token.Asset.GetInfo(reqData)
 if resData.ErrorCode == 0 {
   data, _ := json.Marshal(resData.Result.Assets)
   fmt.Println("Assets:", string(data))
 }

合约服务
--------

合约服务主要是合约相关的接口，目前有1个接口: ``GetInfo`` 。

GetInfo-contract
~~~~~~~~~~~~~~~~

``GetInfo-contract`` 接口用来获取合约信息。

调用方法如下：

::

 GetInfo(model.ContractGetInfoRequest) model.ContractGetInfoResponse

请求参数如下表：

+-----------------+--------+--------------------+
| 参数            | 类型   | 描述               |
+=================+========+====================+
| contractAddress | string | 必填，合约账户地址 |
+-----------------+--------+--------------------+

响应数据如下表：

+---------+--------+-----------------+
| 参数    | 类型   | 描述            |
+=========+========+=================+
| Type    | int64  | 合约类型，默认0 |
+---------+--------+-----------------+
| Payload | string | 合约代码        |
+---------+--------+-----------------+

错误码如下表：

+-------------------------+------------+------------------+
| 错误信息                | 错误码     | 描述             |
+=========================+============+==================+
| INVALID_CONTRACTADDRESS | 11037      | Invalid contract |
| _ERROR                  |            | address          |
+-------------------------+------------+------------------+
| CONTRACTADDRESS_NOT_CON | 11038      | contractAddress  |
| TRACTACCOUNT_ERROR      |            | is not a         |
|                         |            | contract account |
+-------------------------+------------+------------------+
| CONNECTNETWORK_ERROR    | 11007      | Failed to connect|
|                         |            | to the network   |
+-------------------------+------------+------------------+
| SYSTEM_ERROR            | 20000      | System error     |
+-------------------------+------------+------------------+

具体示例如下所示：

::

 var reqData model.ContractGetInfoRequest
 var address string = "buQfnVYgXuMo3rvCEpKA6SfRrDpaz8D8A9Ea"
 reqData.SetAddress(address)
 resData := testSdk.Contract.GetInfo(reqData)
 if resData.ErrorCode == 0 {
   data, _ := json.Marshal(resData.Result.Contract)
   fmt.Println("Contract:", string(data))
 }

交易服务
--------

交易服务主要是交易相关的接口，目前有5个接口：``EvaluateFee``、``BuildBlob``、
``Sign``、``Submit``、``GetInfo-transaction``。

EvaluateFee
~~~~~~~~~~~

``EvaluateFee`` 接口用来评估交易费用。

调用方法如下:

::

 EvaluateFee(model.TransactionEvaluateFeeRequest)model.TransactionEvaluateFeeResponse

请求参数如下表：

+-----------------------+---------------------+---------------------------------+
| 参数                  | 类型                | 描述                            |
+=======================+=====================+=================================+
| sourceAddress         | string              | 必填，发起该操作的源账户地址    |
+-----------------------+---------------------+---------------------------------+
| nonce                 | int64               | 必填，待发起的交易序列号，      |
|                       |                     | 大小[1,max(int64)]              |
+-----------------------+---------------------+---------------------------------+
| operations            | list.List           | 必填，待提交的操作列表，不能为空|
+-----------------------+---------------------+---------------------------------+
| signatureNumber       | string              | 选填，待签名者的数量，默认是1， |
|                       |                     | 大小[1,max(int32)]              |
+-----------------------+---------------------+---------------------------------+
| metadata              | string              | 选填，备注                      |
+-----------------------+---------------------+---------------------------------+
| ceilLedgerSeq         | int64               | 选填，距离当前区块高度指定差值  |
|                       |                     | 的区块内执行的限制，当区块超出  |
|                       |                     | 当时区块高度与所设差值的和后，  |
|                       |                     | 交易执行失败。必须大于等于0，   |
|                       |                     | 是0时不限制                     |
+-----------------------+---------------------+---------------------------------+

响应数据如下表：

+----------+-------+----------+
| 成员变量 | 类型  | 描述     |
+==========+=======+==========+
| FeeLimit | int64 | 交易费用 |
+----------+-------+----------+
| GasPrice | int64 | 打包费用 |
+----------+-------+----------+

错误码如下表：

+-------------------------+----------+------------------+
| 错误信息                | 错误码   | 描述             |
+=========================+==========+==================+
| INVALID_SOURCEADDRESS   | 11002    | Invalid          |
| _ERROR                  |          | sourceAddress    |
+-------------------------+----------+------------------+
| INVALID_NONCE_ERROR     | 11048    | Nonce must be    |
|                         |          | between 1 and    |
|                         |          | max(int64)       |
+-------------------------+----------+------------------+
| INVALID_OPERATIONS      | 11051    | Operations       |
| _ERROR                  |          | cannot be        |
|                         |          | resolved         |
+-------------------------+----------+------------------+
| OPERATIONS_ONE_ERROR    | 11053    | One of the       |
|                         |          | operations cannot|
|                         |          | be resolved      |
+-------------------------+----------+------------------+
| INVALID_SIGNATURENUMBER | 11054    | SignatureNumber  |
| _ERROR                  |          | must be between  |
|                         |          | 1 and max(int32) |
+-------------------------+----------+------------------+
| SYSTEM_ERROR            | 20000    | System error     |
+-------------------------+----------+------------------+  

具体示例如下所示:

::

   var reqDataOperation model.BUSendOperation
   reqDataOperation.Init()
   var amount int64 = 100
   reqDataOperation.SetAmount(amount)
   var destAddress string = "buQVU86Jm4FeRW4JcQTD9Rx9NkUkHikYGp6z"
   reqDataOperation.SetDestAddress(destAddress)

   var reqDataEvaluate model.TransactionEvaluateFeeRequest
   var sourceAddress string = "buQVU86Jm4FeRW4JcQTD9Rx9NkUkHikYGp6z"
   reqDataEvaluate.SetSourceAddress(sourceAddress)
   var nonce int64 = 88
   reqDataEvaluate.SetNonce(nonce)
   var signatureNumber string = "3"
   reqDataEvaluate.SetSignatureNumber(signatureNumber)
   var SetCeilLedgerSeq int64 = 50
   reqDataEvaluate.SetCeilLedgerSeq(SetCeilLedgerSeq)
   reqDataEvaluate.SetOperation(reqDataOperation)
   resDataEvaluate := testSdk.Transaction.EvaluateFee(reqDataEvaluate)
   if resDataEvaluate.ErrorCode == 0 {
       data, _ := json.Marshal(resDataEvaluate.Result)
       fmt.Println("Evaluate:", string(data))
   }

BuildBlob
~~~~~~~~~

``BuildBlob`` 接口用于序列化交易，生成交易Blob串，便于网络传输。在调用BuildBlob之前需要构建一些操作对象，目前的操作对象有16种,参见 `BaseOperation`_。

调用方法如下：

::
 
 BuildBlob(model.TransactionBuildBlobRequest)model.TransactionBuildBlobResponse

请求参数如下表：

+-----------------------+-----------+---------------------------------+
| 参数                  | 类型      | 描述                            |
+=======================+===========+=================================+
| sourceAddress         | string    | 必填，发起该操作的源账户地址    |
+-----------------------+-----------+---------------------------------+
| nonce                 | int64     | 必填，待发起的交易序列号，      |
|                       |           | 函数里+1，大小[1,max(int64)]    |
+-----------------------+-----------+---------------------------------+
| gasPrice              | int64     | 必填，交易打包费用，单位MO，    |
|                       |           | 1BU = 10^8 MO，大小[1000,       |
|                       |           | max(int64)]                     |
+-----------------------+-----------+---------------------------------+
| feeLimit              | int64     | 必填，交易手续费，单位MO，1     |
|                       |           | BU = 10^8 MO，                  |
|                       |           | 大小[1,max(int64)]              |
+-----------------------+-----------+---------------------------------+
| operations            | list.List | 必填，待提交的操作列表，        |
|                       |           | 不能为空                        |
+-----------------------+-----------+---------------------------------+
| ceilLedgerSeq         | int64     | 选填，距离当前区块高度指定      |
|                       |           | 差值的区块内执行的限制，        |
|                       |           | 当区块超出当时区块高度与        |
|                       |           | 所设差值的和后，交易执行失败。  |
|                       |           | 必须大于等于0，是0时不限制      |            
+-----------------------+-----------+---------------------------------+
| metadata              | string    | 选填，备注                      |
+-----------------------+-----------+---------------------------------+

响应数据如下表：

+-----------------+--------+-----------------------------------+
| 参数            | 类型   | 描述                              |
+=================+========+===================================+
| TransactionBlob | string | Transaction序列化后的16进制字符串 |
+-----------------+--------+-----------------------------------+

错误码如下表：

+-------------------------+------------+------------------+
| 错误信息                | 错误码     | 描述             |
+=========================+============+==================+
| INVALID_SOURCEADDRESS   | 11002      | Invalid          |
| _ERROR                  |            | sourceAddress    |
+-------------------------+------------+------------------+
| INVALID_NONCE_ERROR     | 11048      | Nonce must be    |
|                         |            | between 1 and    |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_DESTADDRESS     | 11003      | Invalid          |
| _ERROR                  |            | destAddress      |
+-------------------------+------------+------------------+
| INVALID_INITBALANCE     | 11004      | InitBalance must |
| _ERROR                  |            | be between 1 and |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| SOURCEADDRESS_EQUAL     | 11005      | SourceAddress    |
| _DESTADDRESS_ERROR      |            | cannot be equal  |
|                         |            | to destAddress   |
+-------------------------+------------+------------------+
| INVALID_ISSUE_AMMOUNT   | 11008      | AssetAmount to   |
| _ERROR                  |            | be issued        |
|                         |            | must be between  |
|                         |            | 1 and max(int64) |
+-------------------------+------------+------------------+
| INVALID_DATAKEY_ERROR   | 11011      | The length of    |
|                         |            | key must be      |
|                         |            | between 1 and    |
|                         |            | 1024             |
+-------------------------+------------+------------------+
| INVALID_DATAVALUE_ERROR | 11012      | The length of    |
|                         |            | value must be    |
|                         |            | between 0 and    |
|                         |            | 256k             |
+-------------------------+------------+------------------+
| INVALID_DATAVERSION     | 11013      | The version must |
| _ERROR                  |            | be greater than  |
|                         |            | or equal to 0    |
+-------------------------+------------+------------------+
| INVALID_MASTERWEIGHT    | 11015      | MasterWeight     |
| _ERROR                  |            | must be between  |
|                         |            | 0 and            |
|                         |            | max(uint32)      |
+-------------------------+------------+------------------+
| INVALID_SIGNER_ADDRESS  | 11016      | Invalid signer   |
| _ERROR                  |            | address          |
+-------------------------+------------+------------------+
| INVALID_SIGNER_WEIGHT   | 11017      | Signer weight    |
| _ERROR                  |            | must be between  |
|                         |            | 0 and            |
|                         |            | max(uint32)      |
+-------------------------+------------+------------------+
| INVALID_TX_THRESHOLD    | 11018      | TxThreshold must |
| _ERROR                  |            | be between 0 and |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_OPERATION_TYPE  | 11019      | Operation type   |
| _ERROR                  |            | must be between  |
|                         |            | 1 and 100        |
+-------------------------+------------+------------------+
| INVALID_TYPE_THRESHOLD  | 11020      | TypeThreshold    |
| _ERROR                  |            | must be between  |
|                         |            | 0 and max(int64) |
+-------------------------+------------+------------------+
| INVALID_ASSET_CODE      | 11023      | The length of    |
| _ERROR                  |            | code must be     |
|                         |            | between 1 and 64 |
+-------------------------+------------+------------------+
| INVALID_ASSET_AMOUNT    | 11024      | AssetAmount must |
| _ERROR                  |            | be between 0 and |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_BU_AMOUNT_ERROR | 11026      | BuAmount must be |
|                         |            | between 0 and    |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_ISSUER_ADDRESS  | 11027      | Invalid issuer   |
| _ERROR                  |            | address          |
+-------------------------+------------+------------------+
| NO_SUCH_TOKEN_ERROR     | 11030      | The length of    |
|                         |            | ctp must be      |
|                         |            | between 1 and 64 |
+-------------------------+------------+------------------+
| INVALID_TOKEN_NAME      | 11031      | The length of    |
| _ERROR                  |            | token name must  |
|                         |            | be between 1 and |
|                         |            | 1024             |
+-------------------------+------------+------------------+
| INVALID_TOKEN_SYMBOL    | 11032      | The length of    |
| _ERROR                  |            | symbol must be   |
|                         |            | between 1 and    |
|                         |            | 1024             |
+-------------------------+------------+------------------+
| INVALID_TOKEN_DECIMALS  | 11033      | Decimals must be |
| _ERROR                  |            | between 0 and 8  |
+-------------------------+------------+------------------+
| INVALID_TOKEN_TOTALSUPP | 11034      | TotalSupply must |
| LY_ERROR                |            | be between 1 and |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_TOKENOWNER      | 11035      | Invalid token    |
| _ERRP                   |            | owner            |
+-------------------------+------------+------------------+
| INVALID_CONTRACTADDRESS | 11037      | Invalid contract |
| _ERROR                  |            | address          |
+-------------------------+------------+------------------+
| CONTRACTADDRESS_NOT     | 11038      | ContractAddress  |
| _CONTRACTACCOUNT_ERRO   |            | is not a         |
|                         |            | contract account |
+-------------------------+------------+------------------+
| INVALID_TOKEN_AMOUNT    | 11039      | Amount           |
| _ERROR                  |            | must be between  |
|                         |            | 1 and max(int64) |
+-------------------------+------------+------------------+
| SOURCEADDRESS_EQUAL     | 11040      | SourceAddress    |
| _CONTRACTADDRESS_ERROR  |            | cannot be equal  |
|                         |            | to               |
|                         |            | contractAddress  |
+-------------------------+------------+------------------+
| INVALID_FROMADDRESS     | 11041      | Invalid          |
| _ERROR                  |            | fromAddress      |
+-------------------------+------------+------------------+
| FROMADDRESS_EQUAL_DESTA | 11042      | FromAddress      |
| DDRESS_ERROR            |            | cannot be equal  |
|                         |            | to destAddress   |
+-------------------------+------------+------------------+
| INVALID_SPENDER_ERROR   | 11043      | Invalid spender  |
+-------------------------+------------+------------------+
| PAYLOAD_EMPTY_ERROR     | 11044      | Payload cannot   |
|                         |            | be empty         |
+-------------------------+------------+------------------+
| INVALID_LOG_TOPIC       | 11045      | The length of    |
| _ERROR                  |            | log topic must   |
|                         |            | be between 1     |
|                         |            | and 128          |
+-------------------------+------------+------------------+
| INVALID_LOG_DATA        | 11046      | The length of    |
| _ERROR                  |            | log data must be |
|                         |            | between 1 and    |
|                         |            | 1024             |
+-------------------------+------------+------------------+
| INVALID_CONTRACT_TYPE   | 11047      | Type must be     |
| _ERROR                  |            | greater than or  |
|                         |            | equal to 0       |
+-------------------------+------------+------------------+
| INVALID_NONCE_ERROR     | 11048      | Nonce must be    |
|                         |            | between 1 and    |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_GASPRICE        | 11049      | GasPrice must be |
| _ERROR                  |            | between 1000 and |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| INVALID_FEELIMIT_ERROR  | 11050      | FeeLimit must be |
|                         |            | between 1 and    |
|                         |            | max(int64)       |
+-------------------------+------------+------------------+
| OPERATIONS_EMPTY_ERROR  | 11051      | Operations       |
|                         |            | cannot be empty  |
+-------------------------+------------+------------------+
| INVALID_CEILLEDGERSEQ   | 11052      | CeilLedgerSeq    |
| _ERROR                  |            | must be equal or |
|                         |            | greater than 0   |
+-------------------------+------------+------------------+
| OPERATIONS_ONE_ERROR    | 11053      | One of the       |
|                         |            | operations       |
|                         |            | cannot be        |
|                         |            | resolved         |
+-------------------------+------------+------------------+
| SYSTEM_ERROR            | 20000      | System error     |
+-------------------------+------------+------------------+

具体示例如下所示:

::

   var reqDataOperation model.BUSendOperation
   reqDataOperation.Init()
   var amount int64 = 100
   var destAddress string = "buQVU86Jm4FeRW4JcQTD9Rx9NkUkHikYGp6z"
   reqDataOperation.SetAmount(amount)
   reqDataOperation.SetDestAddress(destAddress)

   var reqDataBlob model.TransactionBuildBlobRequest
   var sourceAddressBlob string = "buQemmMwmRQY1JkcU7w3nhruoX5N3j6C29uo"
   reqDataBlob.SetSourceAddress(sourceAddressBlob)
   var feeLimit int64 = 1000000000
   reqDataBlob.SetFeeLimit(feeLimit)
   var gasPrice int64 = 1000
   reqDataBlob.SetGasPrice(gasPrice)
   var nonce int64 = 88
   reqDataBlob.SetNonce(nonce)
   reqDataBlob.SetOperation(reqDataOperation)

   resDataBlob := testSdk.Transaction.BuildBlob(reqDataBlob)
   if resDataBlob.ErrorCode == 0 {
       fmt.Println("Blob:", resDataBlob.Result)
   }



BaseOperation
^^^^^^^^^^^^^

在调用BuildBlob之前需要构建一些操作对象，目前的操作对象有16种: ``AccountActivateOperation``、``AccountSetMetadataOperation``、``AccountSetPrivilegeOperation``、
``AssetIssueOperation``、``AssetSendOperation``、 ``BUSendOperation``、``Ctp10TokenIssueOperation``、
``Ctp10TokenTransferOperation``、``Ctp10TokenTransferFromOperation``、``Ctp10TokenApproveOperation``、
``Ctp10TokenAssignOperation``、``Ctp10TokenChangeOwnerOperation``、``ContractCreateOperation``、
``ContractInvokeByAssetOperation``、``ContractInvokeByBUOperation``、``LogCreateOperation``。

AccountActivateOperation

+---------------+--------+---------------------------------------+
| 成员变量      | 类型   | 描述                                  |
+===============+========+=======================================+
| sourceAddress | string | 选填，操作源账户                      |
+---------------+--------+---------------------------------------+
| destAddress   | string | 必填，目标账户地址                    |
+---------------+--------+---------------------------------------+
| initBalance   | int64  | 必填，初始化资产，大小[1, max(int64)] |
+---------------+--------+---------------------------------------+
| metadata      | string | 选填，备注                            |
+---------------+--------+---------------------------------------+

AccountSetMetadataOperation

+---------------+--------+---------------------------------------+
| 成员变量      | 类型   | 描述                                  |
+===============+========+=======================================+
| sourceAddress | string | 选填，操作源账户                      |
+---------------+--------+---------------------------------------+
| key           | string | 必填，metadata的关键词，长度[1, 1024] |
+---------------+--------+---------------------------------------+
| value         | string | 选填，metadata的内容，长度[0, 256K]   |
+---------------+--------+---------------------------------------+
| version       | int64  | 选填，metadata的版本                  |
+---------------+--------+---------------------------------------+
| deleteFlag    | bool   | 选填，是否删除metadata                |
+---------------+--------+---------------------------------------+
| metadata      | string | 选填，备注                            |
+---------------+--------+---------------------------------------+

AccountSetPrivilegeOperation

+-----------------------+-----------------------+-----------------------+
| 成员变量              | 类型                  | 描述                  |
+=======================+=======================+=======================+
| sourceAddress         | string                | 选填，操作源账户      |
+-----------------------+-----------------------+-----------------------+
| masterWeight          | string                | 选填，账户自身权重，  |
|                       |                       | 大小[0, max(uint32)]  |
+-----------------------+-----------------------+-----------------------+
| signers               | [] `Signer`_          | 选填，签名者权重列表  |
+-----------------------+-----------------------+-----------------------+
| txThreshold           | string                | 选填，交易门限，      |
|                       |                       | 大小[0,max(int64)]    |
+-----------------------+-----------------------+-----------------------+
| typeThreshold         | `TypeThreshold`_      | 选填，指定类型交易门限|
+-----------------------+-----------------------+-----------------------+
| metadata              | string                | 选填，备注            |
+-----------------------+-----------------------+-----------------------+

AssetIssueOperation

+---------------+--------+-----------------------------------------+
| 成员变量      | 类型   | 描述                                    |
+===============+========+=========================================+
| sourceAddress | string | 选填，发起该操作的源账户地址            |
+---------------+--------+-----------------------------------------+
| code          | string | 必填，资产编码，长度[1 64]              |
+---------------+--------+-----------------------------------------+
| amount        | int64  | 必填，资产发行数量，大小[1, max(int64)] |
+---------------+--------+-----------------------------------------+
| metadata      | string | 选填，备注                              |
+---------------+--------+-----------------------------------------+

AssetSendOperation

+---------------+--------+--------------------------------------+
| 成员变量      | 类型   | 描述                                 |
+===============+========+======================================+
| sourceAddress | string | 选填，发起该操作的源账户地址         |
+---------------+--------+--------------------------------------+
| destAddress   | string | 必填，目标账户地址                   |
+---------------+--------+--------------------------------------+
| code          | string | 必填，资产编码，长度[1 64]           |
+---------------+--------+--------------------------------------+
| issuer        | string | 必填，资产发行账户地址               |
+---------------+--------+--------------------------------------+
| amount        | int64  | 必填，资产数量，大小[ 0, max(int64)] |
+---------------+--------+--------------------------------------+
| metadata      | string | 选填，备注                           |
+---------------+--------+--------------------------------------+

BUSendOperation

+---------------+--------+-----------------------------------------+
| 成员变量      | 类型   | 描述                                    |
+===============+========+=========================================+
| sourceAddress | string | 选填，发起该操作的源账户地址            |
+---------------+--------+-----------------------------------------+
| destAddress   | string | 必填，目标账户地址                      |
+---------------+--------+-----------------------------------------+
| amount        | int64  | 必填，资产发行数量，大小[0, max(int64)] |
+---------------+--------+-----------------------------------------+
| metadata      | string | 选填，备注                              |
+---------------+--------+-----------------------------------------+

Ctp10TokenIssueOperation

+---------------+--------+---------------------------------------------------+
| 成员变量      | 类型   | 描述                                              |
+===============+========+===================================================+
| sourceAddress | string | 选填，发起该操作的源账户地址                      |
+---------------+--------+---------------------------------------------------+
| initBalance   | int64  | 必填，给合约账户的初始化资产，大小[1, max(int64)] |
+---------------+--------+---------------------------------------------------+
| name          | string | 必填，token名称，长度[1, 1024]                    |
+---------------+--------+---------------------------------------------------+
| symbol        | string | 必填，token符号，长度[1, 1024]                    |
+---------------+--------+---------------------------------------------------+
| decimals      | int64  | 必填，token数量的精度，大小[0, 8]                 |
+---------------+--------+---------------------------------------------------+
| supply        | int64  | 必填，token发行的总供应量，大小[1, max(int64)]    |
+---------------+--------+---------------------------------------------------+
| metadata      | string | 选填，备注                                        |
+---------------+--------+---------------------------------------------------+

Ctp10TokenTransferOperation

+-----------------+--------+----------------------------------------------+
| 成员变量        | 类型   | 描述                                         |
+=================+========+==============================================+
| sourceAddress   | string | 选填，发起该操作的源账户地址                 |
+-----------------+--------+----------------------------------------------+
| contractAddress | string | 必填，合约账户地址                           |
+-----------------+--------+----------------------------------------------+
| destAddress     | string | 必填，待转移的目标账户地址                   |
+-----------------+--------+----------------------------------------------+
| amount          | int64  | 必填，待转移的token数量，大小[1, max(int64)] |
+-----------------+--------+----------------------------------------------+
| metadata        | string | 选填，备注                                   |
+-----------------+--------+----------------------------------------------+

Ctp10TokenTransferFromOperation

+-----------------+--------+----------------------------------------------+
| 成员变量        | 类型   | 描述                                         |
+=================+========+==============================================+
| sourceAddress   | string | 选填，发起该操作的源账户地址                 |
+-----------------+--------+----------------------------------------------+
| contractAddress | string | 必填，合约账户地址                           |
+-----------------+--------+----------------------------------------------+
| fromAddress     | string | 必填，待转移的源账户地址                     |
+-----------------+--------+----------------------------------------------+
| destAddress     | string | 必填，待转移的目标账户地址                   |
+-----------------+--------+----------------------------------------------+
| amount          | int64  | 必填，待转移的token数量，大小[1, max(int64)] |
+-----------------+--------+----------------------------------------------+
| metadata        | string | 选填，备注                                   |
+-----------------+--------+----------------------------------------------+

Ctp10TokenApproveOperation

+-----------------------+-----------------------+-----------------------+
| 成员变量              | 类型                  | 描述                  |
+=======================+=======================+=======================+
| sourceAddress         | string                | 选填，发起该操作的    |
|                       |                       | 源账户地址            |
+-----------------------+-----------------------+-----------------------+
| contractAddress       | string                | 必填，合约账户地址    |
+-----------------------+-----------------------+-----------------------+
| spender               | string                | 必填，授权的账户地址  |
+-----------------------+-----------------------+-----------------------+
| amount                | int64                 | 必填，被授权的        |
|                       |                       | 待转移的token数量，   |
|                       |                       | 大小[1,max(int64)]    |
+-----------------------+-----------------------+-----------------------+
| metadata              | string                | 选填，备注            |
+-----------------------+-----------------------+-----------------------+

Ctp10TokenAssignOperation

+-----------------+--------+----------------------------------------------+
| 成员变量        | 类型   | 描述                                         |
+=================+========+==============================================+
| sourceAddress   | string | 选填，发起该操作的源账户地址                 |
+-----------------+--------+----------------------------------------------+
| contractAddress | string | 必填，合约账户地址                           |
+-----------------+--------+----------------------------------------------+
| destAddress     | string | 必填，待分配的目标账户地址                   |
+-----------------+--------+----------------------------------------------+
| amount          | int64  | 必填，待分配的token数量，大小[1, max(int64)] |
+-----------------+--------+----------------------------------------------+
| metadata        | string | 选填，备注                                   |
+-----------------+--------+----------------------------------------------+

Ctp10TokenChangeOwnerOperation

+-----------------+--------+------------------------------+
| 成员变量        | 类型   | 描述                         |
+=================+========+==============================+
| sourceAddress   | string | 选填，发起该操作的源账户地址 |
+-----------------+--------+------------------------------+
| contractAddress | string | 必填，合约账户地址           |
+-----------------+--------+------------------------------+
| tokenOwner      | string | 必填，待分配的目标账户地址   |
+-----------------+--------+------------------------------+
| metadata        | string | 选填，备注                   |
+-----------------+--------+------------------------------+

ContractCreateOperation

+---------------+--------+---------------------------------------------------+
| 成员变量      | 类型   | 描述                                              |
+===============+========+===================================================+
| sourceAddress | string | 选填，发起该操作的源账户地址                      |
+---------------+--------+---------------------------------------------------+
| initBalance   | int64  | 必填，给合约账户的初始化资产，大小[1, max(int64)] |
+---------------+--------+---------------------------------------------------+
| initInput     | string | 选填，对应的合约初始化参数                        |
+---------------+--------+---------------------------------------------------+
| payload       | string | 必填，对应的合约代码                              |
+---------------+--------+---------------------------------------------------+
| metadata      | string | 选填，备注                                        |
+---------------+--------+---------------------------------------------------+

ContractInvokeByAssetOperation

+------------------+----------+-----------------------+
| 成员变量         | 类型     | 描述                  |
+==================+==========+=======================+
| sourceAddress    | string   | 选填，发起该操作的    |
|                  |          | 源账户地址            |
+------------------+----------+-----------------------+
| contractAddress  | string   | 必填，合约账户地址    |
+------------------+----------+-----------------------+
| code             | string   | 选填，资产编码，长    |
|                  |          | 度[0,64]，当为null时，|
|                  |          | 仅触发合约            |
+------------------+----------+-----------------------+
| issuer           | string   | 选填，资产发行账户    |
|                  |          | 地址，当为null时，    |
|                  |          | 仅触发合约            |
+------------------+----------+-----------------------+
| amount           | int64    | 选填，资产数量，      |
|                  |          | 大小[0,max(int64)]，  |
|                  |          | 当是0时，仅触发合约   |
+------------------+----------+-----------------------+
| input            | string   | 选填，待触发的合约的  |
|                  |          | main()入参            |
+------------------+----------+-----------------------+
| metadata         | string   | 选填，备注            |
+------------------+----------+-----------------------+

ContractInvokeByBUOperation

+--------------------+----------+--------------------------------+
| 成员变量           | 类型     | 描述                           |
+====================+==========+================================+
| sourceAddress      | string   | 选填，发起该操作的源账户地址   |
+--------------------+----------+--------------------------------+
| contractAddress    | string   | 必填，合约账户地址             |
+--------------------+----------+--------------------------------+
| amount             | int64    | 选填，资产发行数量，           |
|                    |          | 大小[0,max(int64)]，           |
|                    |          | 当0时仅触发合约                |
+--------------------+----------+--------------------------------+
| input              | string   | 选填，待触发的合约的main()入参 |
+--------------------+----------+--------------------------------+
| metadata           | string   | 选填，备注                     |
+--------------------+----------+--------------------------------+

LogCreateOperation

+---------------+----------+-----------------------------------------+
| 成员变量      | 类型     | 描述                                    |
+===============+==========+=========================================+
| sourceAddress | string   | 选填，发起该操作的源账户地址            |
+---------------+----------+-----------------------------------------+
| topic         | string   | 必填，日志主题，长度[1, 128]            |
+---------------+----------+-----------------------------------------+
| data          | []string | 必填，日志内容，每个字符串长度[1, 1024] |
+---------------+----------+-----------------------------------------+
| metadata      | string   | 选填，备注                              |
+---------------+----------+-----------------------------------------+

Sign
~~~~

``Sign`` 接口用于实现交易的签名。

调用方法如下：

::

 Sign(model.TransactionSignRequest) model.TransactionSignResponse

请求参数如下表：

+-------------+-----------+------------------------+
| 参数        | 类型      | 描述                   |
+=============+===========+========================+
| blob        | string    | 必填，待签名的交易Blob |
+-------------+-----------+------------------------+
| privateKeys | [] string | 必填，私钥列表         |
+-------------+-----------+------------------------+

响应数据如下表：

+------------+------------------+------------------+
| 参数       | 类型             | 描述             |
+============+==================+==================+
| Signatures | [] `signature`_  | 签名后的数据列表 |
+------------+------------------+------------------+

错误码如下表：

+------------------------+--------+---------------------------------------+
| 错误信息               | 错误码 | 描述                                  |
+========================+========+=======================================+
| INVALID_BLOB_ERROR     | 11056  | Invalid blob                          |
+------------------------+--------+---------------------------------------+
| PRIVATEKEY_NULL_ERROR  | 11057  | PrivateKeys cannot be empty           |
+------------------------+--------+---------------------------------------+
| PRIVATEKEY_ONE_ERROR   | 11058  | One of privateKeys error              |
+------------------------+--------+---------------------------------------+
| GET_ENCPUBLICKEY_ERROR | 14000  | The function ‘GetEncPublicKey’ failed |
+------------------------+--------+---------------------------------------+
| SIGN_ERROR             | 14001  | The function ‘Sign’ failed            |
+------------------------+--------+---------------------------------------+
| SYSTEM_ERROR           | 20000  | System error                          |
+------------------------+--------+---------------------------------------+

具体示例如下所示:

::

   PrivateKey := []string{"privbUPxs6QGkJaNdgWS2hisny6ytx1g833cD7V9C3YET9mJ25wdcq6h"}
   var reqData model.TransactionSignRequest
   reqData.SetBlob(resDataBlob.Result.Blob)
   reqData.SetPrivateKeys(PrivateKey)
   resDataSign := testSdk.Transaction.Sign(reqData)
   if resDataSign.ErrorCode == 0 {
       fmt.Println("Sign:", resDataSign.Result)
   }

接口对象类型参考
^^^^^^^^^^^^^^^

下面是 ``Sign`` 的接口对象类型：

Signature
+++++++++

+-----------+-------+------------+
| 成员变量  | 类型  | 描述       |
+===========+=======+============+
| signData  | int64 | 签名后数据 |
+-----------+-------+------------+
| publicKey | int64 | 公钥       |
+-----------+-------+------------+


Submit
~~~~~~

``Submit`` 接口用于提交交易。

调用方法如下：

::
 
 Submit(model.TransactionSubmitRequest) model.TransactionSubmitResponse

请求参数如下表：

+-----------+-------------------+----------------+
| 参数      | 类型              | 描述           |
+===========+===================+================+
| blob      | string            | 必填，交易blob |
+-----------+-------------------+----------------+
| signature | [] `signature`_   | 必填，签名列表 |
+-----------+-------------------+----------------+

响应数据如下表：

+------+--------+----------+
| 参数 | 类型   | 描述     |
+======+========+==========+
| Hash | string | 交易hash |
+------+--------+----------+

错误码如下表：

+--------------------+--------+--------------+
| 错误信息           | 错误码 | 描述         |
+====================+========+==============+
| INVALID_BLOB_ERROR | 11052  | Invalid blob |
+--------------------+--------+--------------+
| SYSTEM_ERROR       | 20000  | System error |
+--------------------+--------+--------------+

具体示例如下所示：

::

   var reqData model.TransactionSubmitRequest
   reqData.SetBlob(resDataBlob.Result.Blob)
   reqData.SetSignatures(resDataSign.Result.Signatures)
   resDataSubmit := testSdk.Transaction.Submit(reqData.Result)
   if resDataSubmit.ErrorCode == 0 {
       fmt.Println("Hash:", resDataSubmit.Result.Hash)
   }

GetInfo-transaction
~~~~~~~~~~~~~~~~~~~~

``GetInfo-transaction`` 接口用于根据hash查询交易。

调用方法如下：

::

 GetInfo(model.TransactionGetInfoRequest)model.TransactionGetInfoResponse

请求参数如下表：

+------+--------+----------+
| 参数 | 类型   | 描述     |
+======+========+==========+
| hash | string | 交易hash |
+------+--------+----------+

响应数据如下表：

+-----------------------+------------------------------+-----------------------+
| 参数                  | 类型                         | 描述                  |
+=======================+==============================+=======================+
| TotalCount            | int64                        | 返回的总交易数        |
+-----------------------+------------------------------+-----------------------+
| Transactions          | [] `TransactionHistory`_     | 交易内容              |
+-----------------------+------------------------------+-----------------------+


具体示例如下所示：

::

   var reqData model.TransactionGetInfoRequest
   var hash string = "cd33ad1e033d6dfe3db3a1d29a55e190935d9d1ff40a138d777e9406ebe0fdb1"
   reqData.SetHash(hash)
   resData := testSdk.Transaction.GetInfo(reqData)
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result)
       fmt.Println("info:", string(data)
   }

接口对象类型参考
^^^^^^^^^^^^^^^^

下面是 ``GetInfo-transaction`` 的接口对象类型：

TransactionHistory
++++++++++++++++++

+--------------+---------------------+--------------+
| 成员变量     | 类型                | 描述         |
+==============+=====================+==============+
| ActualFee    | string              | 交易实际费用 |
+--------------+---------------------+--------------+
| CloseTime    | int64               | 交易关闭时间 |
+--------------+---------------------+--------------+
| ErrorCode    | int64               | 交易错误码   |
+--------------+---------------------+--------------+
| ErrorDesc    | string              | 交易描述     |
+--------------+---------------------+--------------+
| Hash         | string              | 交易hash     |
+--------------+---------------------+--------------+
| LedgerSeq    | int64               | 区块序列号   |
+--------------+---------------------+--------------+
| Transactions | `Transaction`_      | 交易内容列表 |
+--------------+---------------------+--------------+
| Signatures   | [] `Signature`_     | 签名列表     |
+--------------+---------------------+--------------+
| TxSize       | int64               | 交易大小     |
+--------------+---------------------+--------------+

Transaction
++++++++++++

+---------------+-------------------+----------------------+
| 成员          | 类型              | 描述                 |
+===============+===================+======================+
| SourceAddress | string            | 交易发起的源账户地址 |
+---------------+-------------------+----------------------+
| FeeLimit      | int64             | 交易费用             |
+---------------+-------------------+----------------------+
| GasPrice      | int64             | 交易打包费用         |
+---------------+-------------------+----------------------+
| Nonce         | int64             | 交易序列号           |
+---------------+-------------------+----------------------+
| Operations    | []  `Operation`_  | 操作列表             |
+---------------+-------------------+----------------------+

Operation
++++++++++

+---------------+--------------------+--------------------+
| 成员          | 类型               | 描述               |
+===============+====================+====================+
| Type          | int64              | 操作类型           |
+---------------+--------------------+--------------------+
| SourceAddress | string             | 操作发起源账户地址 |
+---------------+--------------------+--------------------+
| Metadata      | string             | 备注               |
+---------------+--------------------+--------------------+
| CreateAccount | `CreateAccount`_   | 创建账户操作       |
+---------------+--------------------+--------------------+
| IssueAsset    | `IssueAsset`_      | 发行资产操作       |
+---------------+--------------------+--------------------+
| PayAsset      | `PayAsset`_        | 转移资产操作       |
+---------------+--------------------+--------------------+
| PayCoin       | `PayCoin`_         | 发送BU操作         |
+---------------+--------------------+--------------------+
| SetMetadata   | `SetMetadata`_     | 设置metadata操作   |
+---------------+--------------------+--------------------+
| SetPrivilege  | `SetPrivilege`_    | 设置账户权限操作   |
+---------------+--------------------+--------------------+
| Log           | `Log`_             | 记录日志           |
+---------------+--------------------+--------------------+

TriggerTransaction
+++++++++++++++++++

+------+--------+----------+
| 成员 | 类型   | 描述     |
+======+========+==========+
| hash | string | 交易hash |
+------+--------+----------+

CreateAccount
++++++++++++++

+-------------+----------------------+--------------------+
| 成员        | 类型                 | 描述               |
+=============+======================+====================+
| DestAddress | string               | 目标账户地址       |
+-------------+----------------------+--------------------+
| Contract    | `Contract`_          | 合约信息           |
+-------------+----------------------+--------------------+
| Priv        | `Priv`_              | 账户权限           |
+-------------+----------------------+--------------------+
| Metadata    | [] :ref:`Metadata-2` | 账户               |
+-------------+----------------------+--------------------+
| InitBalance | int64                | 账户资产           |
+-------------+----------------------+--------------------+
| InitInput   | string               | 合约init函数的入参 |
+-------------+----------------------+--------------------+

Contract
+++++++++

+---------+--------+------------------------+
| 成员    | 类型   | 描述                   |
+=========+========+========================+
| Type    | int64  | 合约的语种，默认不赋值 |
+---------+--------+------------------------+
| Payload | string | 对应语种的合约代码     |
+---------+--------+------------------------+

.. _Metadata-2:

Metadata
++++++++

+---------+--------+------------------+
| 成员    | 类型   | 描述             |
+=========+========+==================+
| Key     | string | metadata的关键词 |
+---------+--------+------------------+
| Value   | string | metadata的内容   |
+---------+--------+------------------+
| Version | int64  | metadata的版本   |
+---------+--------+------------------+

IssueAsset
+++++++++++

+--------+--------+----------------------+
| 成员   | 类型   | 描述                 |
+========+========+======================+
| Code   | string | 资产编码，长度[1 64] |
+--------+--------+----------------------+
| Amount | int64  | 资产数量             |
+--------+--------+----------------------+

PayAsset
+++++++++

+-------------+-----------+----------------------+
| 成员        | 类型      | 描述                 |
+=============+===========+======================+
| DestAddress | string    | 待转移的目标账户地址 |
+-------------+-----------+----------------------+
| Asset       | `Asset`_  | 账户资产             |
+-------------+-----------+----------------------+
| Input       | string    | 合约main函数入参     |
+-------------+-----------+----------------------+

PayCoin
++++++++

+-------------+--------+----------------------+
| 成员        | 类型   | 描述                 |
+=============+========+======================+
| DestAddress | string | 待转移的目标账户地址 |
+-------------+--------+----------------------+
| Amount      | int64  | 待转移的BU数量       |
+-------------+--------+----------------------+
| Input       | string | 合约main函数入参     |
+-------------+--------+----------------------+

SetMetadata
++++++++++++

+------------+--------+------------------+
| 成员       | 类型   | 描述             |
+============+========+==================+
| Key        | string | metadata的关键词 |
+------------+--------+------------------+
| Value      | string | metadata的内容   |
+------------+--------+------------------+
| Version    | int64  | metadata的版本   |
+------------+--------+------------------+
| DeleteFlag | bool   | 是否删除metadata |
+------------+--------+------------------+

SetPrivilege
+++++++++++++

+----------------+-------------------+-----------------------+
| 成员           | 类型              | 描述                  |
+================+===================+=======================+
| MasterWeight   | string            | 账户自身权重，大小[0, |
|                |                   | max(uint32)]          |
+----------------+-------------------+-----------------------+
| Signers        | [] `Signer`_      | 签名者权重列表        |
+----------------+-------------------+-----------------------+
| TxThreshold    | string            | 交易门限，大小[0,     |
|                |                   | max(int64)]           |
+----------------+-------------------+-----------------------+
| TypeThreshold  | `TypeThreshold`_  | 指定类型交易门限      |
+----------------+-------------------+-----------------------+

Log
++++

+-------+----------+----------+
| 成员  | 类型     | 描述     |
+=======+==========+==========+
| Topic | string   | 日志主题 |
+-------+----------+----------+
| Data  | []string | 日志内容 |
+-------+----------+----------+


区块服务
--------

区块服务主要是区块相关的接口，目前有11个接口：``GetNumber``、``CheckStatus``、``GetTransactions``、
``GetInfo-block``、``GetLatest``、``GetValidators``、``GetLatestValidators``、``GetReward``、``GetLatestReward``、
``GetFees``、``GetLatestFees``。

GetNumber
~~~~~~~~~~~

``GetNumber`` 接口用于获取区块高度。

调用方法如下：

::

 GetNumber() model.BlockGetNumberResponse 

响应数据如下表：

+-------------+-------+---------------------------------+
| 参数        | 类型  | 描述                            |
+=============+=======+=================================+
| BlockNumber | int64 | 最新的区块高度，对应底层字段seq |
+-------------+-------+---------------------------------+

错误码如下表：

+----------------------+--------+-------------------------+
| 错误信息             | 错误码 | 描述                    |
+======================+========+=========================+
| CONNECTNETWORK_ERROR | 11007  | Failed to connect to    |
|                      |        | the network             |
+----------------------+--------+-------------------------+
| SYSTEM_ERROR         | 20000  | System error            |
+----------------------+--------+-------------------------+

具体示例如下所示：

::

   resData := testSdk.Block.GetNumber()
   if resData.ErrorCode == 0 {
       fmt.Println("BlockNumber:", resData.Result.BlockNumber)
   }

CheckStatus
~~~~~~~~~~~~

``CheckStatus`` 接口用于检查区块同步。

调用方法如下：

::

 CheckStatus() model.BlockCheckStatusResponse

响应数据如下表：

+---------------+------+--------------+
| 参数          | 类型 | 描述         |
+===============+======+==============+
| IsSynchronous | bool | 区块是否同步 |
+---------------+------+--------------+

错误码如下表：

+----------------------+--------+-------------------------+
| 错误信息             | 错误码 | 描述                    |
+======================+========+=========================+
| CONNECTNETWORK_ERROR | 11007  | Failed to connect to    |
|                      |        | the network             |
+----------------------+--------+-------------------------+
| SYSTEM_ERROR         | 20000  | System error            |
+----------------------+--------+-------------------------+

具体示例如下所示：

::

   resData := testSdk.Block.CheckStatus()
   if resData.ErrorCode == 0 {
       fmt.Println("IsSynchronous:", resData.Result.IsSynchronous)
   }

GetTransactions
~~~~~~~~~~~~~~~~

``GetTransactions`` 接口用于根据高度查询交易。

调用方法如下：

::

 GetTransactions(model.BlockGetTransactionRequest)model.BlockGetTransactionResponse

请求参数如下表：

+-------------+-------+------------------------+
| 参数        | 类型  | 描述                   |
+=============+=======+========================+
| blockNumber | int64 | 必填，待查询的区块高度 |
+-------------+-------+------------------------+

响应数据如下表:

+-----------------------+------------------------------+-----------------+
| 参数                  | 类型                         | 描述            |
+=======================+==============================+=================+
| TotalCount            | int64                        | 返回的总交易数  |
+-----------------------+------------------------------+-----------------+
| Transactions          | [] `TransactionHistory`_     | 交易内容        |
+-----------------------+------------------------------+-----------------+

错误码如下表：

+---------------------------+--------+-------------------------+
| 错误信息                  | 错误码 | 描述                    |
+===========================+========+=========================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must be     |
|                           |        | greater than 0          |
+---------------------------+--------+-------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect       |
|                           |        | to the network          |
+---------------------------+--------+-------------------------+
| SYSTEM_ERROR              | 20000  | System error            |
+---------------------------+--------+-------------------------+ 

具体示例如下所示：

::

   var reqData model.BlockGetTransactionRequest
   var blockNumber int64 = 581283
   reqData.SetBlockNumber(blockNumber)
   resData := testSdk.Block.GetTransactions(reqData)
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Transactions)
       fmt.Println("Transactions:", string(data))
   }

GetInfo-block
~~~~~~~~~~~~~~

``GetInfo-block`` 接口用于获取区块信息。

调用方法如下：

::

 GetInfo(model.BlockGetInfoRequest) model.BlockGetInfoResponse

请求参数如下表：

+-------------+-------+------------------+
| 参数        | 类型  | 描述             |
+=============+=======+==================+
| blockNumber | int64 | 待查询的区块高度 |
+-------------+-------+------------------+

响应数据如下表：

+-----------+--------+--------------+
| 参数      | 类型   | 描述         |
+===========+========+==============+
| CloseTime | int64  | 区块关闭时间 |
+-----------+--------+--------------+
| Number    | int64  | 区块高度     |
+-----------+--------+--------------+
| TxCount   | int64  | 交易总量     |
+-----------+--------+--------------+
| Version   | string | 区块版本     |
+-----------+--------+--------------+

错误码如下表：

+---------------------------+--------+------------------------------------+
| 错误信息                  | 错误码 | 描述                               |
+===========================+========+====================================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must be greater than 0 |
+---------------------------+--------+------------------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect to               |
|                           |        | the network                        |
+---------------------------+--------+------------------------------------+
| SYSTEM_ERROR              | 20000  | System error                       |
+---------------------------+--------+------------------------------------+      

具体示例如下所示:

::

   var reqData model.BlockGetInfoRequest
   var blockNumber int64 = 581283
   reqData.SetBlockNumber(blockNumber)
   resData := testSdk.Block.GetInfo(reqData)
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Header)
       fmt.Println("Header:", string(data))
   }

GetLatest
~~~~~~~~~~

``GetLatest`` 接口用于获取最新区块信息。

调用方法如下所示:

::

 GetLatest() model.BlockGetLatestResponse

响应数据如下表:

+-----------+--------+--------------+
| 参数      | 类型   | 描述         |
+===========+========+==============+
| CloseTime | int64  | 区块关闭时间 |
+-----------+--------+--------------+
| Number    | int64  | 区块高度     |
+-----------+--------+--------------+
| TxCount   | int64  | 交易总量     |
+-----------+--------+--------------+
| Version   | string | 区块版本     |
+-----------+--------+--------------+

错误码如下表：

+----------------------+--------+-------------------------+
| 错误信息             | 错误码 | 描述                    |
+======================+========+=========================+
| CONNECTNETWORK_ERROR | 11007  | Failed to connect to    |
|                      |        | the network             |
+----------------------+--------+-------------------------+
| SYSTEM_ERROR         | 20000  | System error            |
+----------------------+--------+-------------------------+   

具体示例如下所示：

::

   resData := testSdk.Block.GetLatest()
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Header)
       fmt.Println("Header:", string(data))
   }

GetValidators
~~~~~~~~~~~~~~

``GetValidators`` 接口用于获取指定区块中所有验证节点数。

调用方法如下:

::

 GetValidators(model.BlockGetValidatorsRequest)model.BlockGetValidatorsResponse

请求参数如下表：

+-------------+-------+------------------+
| 参数        | 类型  | 描述             |
+=============+=======+==================+
| blockNumber | int64 | 待查询的区块高度 |
+-------------+-------+------------------+

响应数据如下表:

+------------+----------------------+--------------+
| 参数       | 类型                 | 描述         |
+============+======================+==============+
| validators | [] `ValidatorInfo`_  | 验证节点列表 |
+------------+----------------------+--------------+

错误码如下表：

+---------------------------+--------+--------------------------+
| 错误信息                  | 错误码 | 描述                     |
+===========================+========+==========================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must be      |
|                           |        | greater than 0           |
+---------------------------+--------+--------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect to     |
|                           |        | the network              |
+---------------------------+--------+--------------------------+
| SYSTEM_ERROR              | 20000  | System error             |
+---------------------------+--------+--------------------------+  

具体示例如下所示:

::

   var reqData model.BlockGetValidatorsRequest
   var blockNumber int64 = 581283
   reqData.SetBlockNumber(blockNumber)
   resData := testSdk.Block.GetValidators(reqData)
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Validators)
       fmt.Println("Validators:", string(data))
   }

接口对象类型参考
^^^^^^^^^^^^^^^

下面是 ``GetValidators`` 的接口对象类型：

ValidatorInfo
++++++++++++++

+------------------+--------+--------------+
| 参数             | 类型   | 描述         |
+==================+========+==============+
| Address          | String | 共识节点地址 |
+------------------+--------+--------------+
| PledgeCoinAmount | int64  | 验证节点押金 |
+------------------+--------+--------------+



GetLatestValidators
~~~~~~~~~~~~~~~~~~~~

``GetLatestValidators`` 接口用于获取最新区块中所有验证节点数。

调用方法如下所示:

::

 GetLatestValidators() model.BlockGetLatestValidatorsResponse

响应数据如下表:

+------------+-----------------------+--------------+
| 参数       | 类型                  | 描述         |
+============+=======================+==============+
| validators | [] `ValidatorInfo`_   | 验证节点列表 |
+------------+-----------------------+--------------+

错误码如下表：

+---------------------------+--------+----------------------------+
| 错误信息                  | 错误码 | 描述                       |
+===========================+========+============================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must           |
|                           |        | be greater than 0          |
+---------------------------+--------+----------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect to       |
|                           |        | the network                |
+---------------------------+--------+----------------------------+
| SYSTEM_ERROR              | 20000  | System error               |
+---------------------------+--------+----------------------------+   

具体示例如下所示:

::

   resData := testSdk.Block.GetLatestValidators()
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Validators)
       fmt.Println("Validators:", string(data))
   }

GetReward
~~~~~~~~~~

``GetReward`` 接口用于获取指定区块中的区块奖励和验证节点奖励。

调用方法如下所示:

::

   GetReward(model.BlockGetRewardRequest) model.BlockGetRewardResponse

请求参数如下表：

+-------------+-------+------------------------+
| 参数        | 类型  | 描述                   |
+=============+=======+========================+
| blockNumber | int64 | 必填，待查询的区块高度 |
+-------------+-------+------------------------+

响应数据如下表：

+-----------------------+-------------------------+-------------------+
| 参数                  | 类型                    | 描述              |
+=======================+=========================+===================+
| BlockReward           | int64                   | 区块奖励数        |
+-----------------------+-------------------------+-------------------+
| ValidatorsReward      | [] `ValidatorReward`_   | 验证节点奖励情况  |
+-----------------------+-------------------------+-------------------+


错误码如下表：

+---------------------------+--------+------------------------------------+
| 错误信息                  | 错误码 | 描述                               |
+===========================+========+====================================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must be greater than 0 |
+---------------------------+--------+------------------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect to               |
|                           |        | the network                        |
+---------------------------+--------+------------------------------------+
| SYSTEM_ERROR              | 20000  | System error                       |
+---------------------------+--------+------------------------------------+  

具体示例如下所示:

::

   var reqData model.BlockGetRewardRequest
   var blockNumber int64 = 581283
   reqData.SetBlockNumber(blockNumber)
   resData := testSdk.Block.GetReward(reqData)
   if resData.ErrorCode == 0 {
       fmt.Println("ValidatorsReward:", resData.Result.ValidatorsReward)
   }

接口对象类型参考
^^^^^^^^^^^^^^^


下面是 ``GetReward`` 的接口对象类型：

ValidatorReward
++++++++++++++++

+-----------+--------+--------------+
| 成员变量  | 类型   | 描述         |
+===========+========+==============+
| Validator | String | 验证节点地址 |
+-----------+--------+--------------+
| Reward    | int64  | 验证节点奖励 |
+-----------+--------+--------------+

GetLatestReward
~~~~~~~~~~~~~~~~~

``GetLatestReward`` 接口用于获取最新区块中的区块奖励和验证节点奖励。

调用方法如下所示:

::

 GetLatestReward() model.BlockGetLatestRewardResponse

响应数据如下表:

+-----------------------+-------------------------+-----------------------+
| 参数                  | 类型                    | 描述                  |
+=======================+=========================+=======================+
| BlockReward           | int64                   | 区块奖励数            |
+-----------------------+-------------------------+-----------------------+
| ValidatorsReward      | [] `ValidatorReward`_   | 验证节点奖励情况      |
+-----------------------+-------------------------+-----------------------+

错误码如下表：

+----------------------+--------+-------------------------+
| 错误信息             | 错误码 | 描述                    |
+======================+========+=========================+
| CONNECTNETWORK_ERROR | 11007  | Failed to connect to    |
|                      |        | the network             |
+----------------------+--------+-------------------------+
| SYSTEM_ERROR         | 20000  | System error            |
+----------------------+--------+-------------------------+ 

具体示例如下所示:

::

   resData := testSdk.Block.GetLatestReward()
   if resData.ErrorCode == 0 {
       fmt.Println("ValidatorsReward:", resData.Result.ValidatorsReward)
   }

GetFees
~~~~~~~

``GetFees`` 接口用于获取指定区块中的账户最低资产限制和打包费用。

调用方法如下所示:

::

 GetFees(model.BlockGetFeesRequest) model.BlockGetFeesResponse

请求参数如下表：

+-------------+-------+------------------------+
| 参数        | 类型  | 描述                   |
+=============+=======+========================+
| blockNumber | int64 | 必填，待查询的区块高度 |
+-------------+-------+------------------------+

响应数据如下表:

+------+------------+------+
| 参数 | 类型       | 描述 |
+======+============+======+
| Fees | `Fees`_    | 费用 |
+------+------------+------+

错误码如下表：

+---------------------------+--------+--------------------------------+
| 错误信息                  | 错误码 | 描述                           |
+===========================+========+================================+
| INVALID_BLOCKNUMBER_ERROR | 11060  | BlockNumber must               |
|                           |        | be greater than 0              |
+---------------------------+--------+--------------------------------+
| CONNECTNETWORK_ERROR      | 11007  | Failed to connect to           |
|                           |        | the network                    |
+---------------------------+--------+--------------------------------+
| SYSTEM_ERROR              | 20000  | System error                   |
+---------------------------+--------+--------------------------------+    

具体示例如下所示:

::

   var reqData model.BlockGetFeesRequest
   var blockNumber int64 = 581283
   reqData.SetBlockNumber(blockNumber)
   resData := testSdk.Block.GetFees(reqData)
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Fees)
       fmt.Println("Fees:", string(data))
   }

接口对象类型参考
^^^^^^^^^^^^^^^

下面是 ``GetFees`` 的接口对象类型：

Fees
+++++

+-------------+-------+----------------------------------+
| 成员变量    | 类型  | 描述                             |
+=============+=======+==================================+
| BaseReserve | int64 | 账户最低资产限制                 |
+-------------+-------+----------------------------------+
| GasPrice    | int64 | 打包费用，单位MO，1 BU = 10^8 MO |
+-------------+-------+----------------------------------+


GetLatestFees
~~~~~~~~~~~~~

``GetLatestFees`` 接口用于获取最新区块中的账户最低资产限制和打包费用。

调用方法如下所示:

::

 GetLatestFees() model.BlockGetLatestFeesResponse

响应数据如下表:

+------+------------------+------+
| 参数 | 类型             | 描述 |
+======+==================+======+
| Fees | `fees`_          | 费用 |
+------+------------------+------+

错误码如下表：

+----------------------+--------+-------------------------+
| 错误信息             | 错误码 | 描述                    |
+======================+========+=========================+
| CONNECTNETWORK_ERROR | 11007  | Failed to connect to    |
|                      |        | the network             |
+----------------------+--------+-------------------------+
| SYSTEM_ERROR         | 20000  | System error            |
+----------------------+--------+-------------------------+  

具体示例如下所示:

::

   resData := testSdk.Block.GetLatestFees()
   if resData.ErrorCode == 0 {
       data, _ := json.Marshal(resData.Result.Fees)
       fmt.Println("Fees:", string(data))
   }

错误码
-------

公共错误码信息如下表：

+-------+---------------------------------------------------------------+
| 参数  | 描述                                                          |
+=======+===============================================================+
| 11001 | Create account failed.                                        |
+-------+---------------------------------------------------------------+
| 11002 | Invalid sourceAddress.                                        |
+-------+---------------------------------------------------------------+
| 11003 | Invalid destAddress.                                          |
+-------+---------------------------------------------------------------+
| 11004 | InitBalance must be between 1 and max(int64).                 |
+-------+---------------------------------------------------------------+
| 11005 | SourceAddress cannot be equal to destAddress.                 |
+-------+---------------------------------------------------------------+
| 11006 | Invalid address.                                              |
+-------+---------------------------------------------------------------+
| 11007 | Failed to connect to the network.                             |
+-------+---------------------------------------------------------------+
| 11008 | AssetAmount to be issued must be between 1 and max(int64).    |
+-------+---------------------------------------------------------------+
| 11009 | The account does not have this asset.                         |
+-------+---------------------------------------------------------------+
| 11010 | The account does not have this metadata.                      |
+-------+---------------------------------------------------------------+
| 11011 | The length of key must be between 1 and 1024.                 |
+-------+---------------------------------------------------------------+
| 11012 | The length of value must be between 0 and 256k.               |
+-------+---------------------------------------------------------------+
| 11013 | The version must be greater than or equal to 0.               |
+-------+---------------------------------------------------------------+
| 11015 | MasterWeight must be between 0 and max(uint32).               |
+-------+---------------------------------------------------------------+
| 11016 | Invalid signer address.                                       |
+-------+---------------------------------------------------------------+
| 11017 | Signer weight must be between 0 and max(uint32).              |
+-------+---------------------------------------------------------------+
| 11018 | TxThreshold must be between 0 and max(int64).                 |
+-------+---------------------------------------------------------------+
| 11019 | Type of TypeThreshold is invalid.                             |
+-------+---------------------------------------------------------------+
| 11020 | TypeThreshold must be between 0 and max(int64).               |
+-------+---------------------------------------------------------------+
| 11023 | The length of code must be between 1 and 64.                  |
+-------+---------------------------------------------------------------+
| 11024 | AssetAmount must be between 0 and max(int64).                 |
+-------+---------------------------------------------------------------+
| 11026 | BuAmount must be between 0 and max(int64).                    |
+-------+---------------------------------------------------------------+
| 11027 | Invalid issuer address.                                       |
+-------+---------------------------------------------------------------+
| 11030 | The length of ctp must be between 1 and 64.                   |
+-------+---------------------------------------------------------------+
| 11031 | The length of token name must be between 1 and 1024.          |
+-------+---------------------------------------------------------------+
| 11032 | The length of symbol must be between 1 and 1024.              |
+-------+---------------------------------------------------------------+
| 11033 | Decimals must be between 0 and 8.                             |
+-------+---------------------------------------------------------------+
| 11034 | TotalSupply must be between 1 and max(int64).                 |
+-------+---------------------------------------------------------------+
| 11035 | Invalid token owner.                                          |
+-------+---------------------------------------------------------------+
| 11036 | Failed to get allowance.                                      |
+-------+---------------------------------------------------------------+
| 11037 | Invalid contract address.                                     |
+-------+---------------------------------------------------------------+
| 11038 | contractAddress is not a contract account.                    |
+-------+---------------------------------------------------------------+
| 11039 | Amount must be between 1 and max(int64).                      |
+-------+---------------------------------------------------------------+
| 11040 | SourceAddress cannot be equal to contractAddress.             |
+-------+---------------------------------------------------------------+
| 11041 | Invalid fromAddress.                                          |
+-------+---------------------------------------------------------------+
| 11042 | FromAddress cannot be equal to destAddress.                   |
+-------+---------------------------------------------------------------+
| 11043 | Invalid spender.                                              |
+-------+---------------------------------------------------------------+
| 11045 | The length of log topic must be between 1 and 128.            |
+-------+---------------------------------------------------------------+
| 11046 | The length of log data must be between 1 and 1024.            |
+-------+---------------------------------------------------------------+
| 11048 | Nonce must be between 1 and max(int64).                       |
+-------+---------------------------------------------------------------+
| 11049 | GasPrice must be between 1000 and max(int64).                 |
+-------+---------------------------------------------------------------+
| 11050 | FeeLimit must be between 1 and max(int64).                    |
+-------+---------------------------------------------------------------+
| 11051 | Operations cannot be empty.                                   |
+-------+---------------------------------------------------------------+
| 11052 | CeilLedgerSeq must be greater than or equal to 0.             |
+-------+---------------------------------------------------------------+
| 11053 | One of the operations cannot be resolved.                     |
+-------+---------------------------------------------------------------+
| 11054 | SignatureNumber must be between 1 and max(int32).             |
+-------+---------------------------------------------------------------+
| 11055 | Invalid transaction hash.                                     |
+-------+---------------------------------------------------------------+
| 11056 | Invalid blob.                                                 |
+-------+---------------------------------------------------------------+
| 11057 | PrivateKeys cannot be empty.                                  |
+-------+---------------------------------------------------------------+
| 11058 | One of the privateKeys is invalid.                            |
+-------+---------------------------------------------------------------+
| 11060 | BlockNumber must be greater than 0.                           |
+-------+---------------------------------------------------------------+
| 11062 | Url cannot be empty.                                          |
+-------+---------------------------------------------------------------+
| 11063 | ContractAddress and code cannot be empty at the same time.    |
+-------+---------------------------------------------------------------+
| 11064 | OptType must be between 0 and 2.                              |
+-------+---------------------------------------------------------------+
| 11065 | Failed to get allowance.                                      |
+-------+---------------------------------------------------------------+
| 11067 | The signatures cannot be empty.                               |
+-------+---------------------------------------------------------------+
| 11066 | Failed to get token info.                                     |
+-------+---------------------------------------------------------------+
| 20000 | System error.                                                 |
+-------+---------------------------------------------------------------+

Go错误码信息如下表：

+--------+----------------------------------------+
| 参数   | 描述                                   |
+========+========================================+
| 14000  | The function `GetEncPublicKey` failed. |                       
+--------+----------------------------------------+
| 14001  | The function `Sign` failed.            |
+--------+----------------------------------------+
| 14002  | The parameter `payload` is invalid.    |
+--------+----------------------------------------+
| 14003  | The query failed.                      |
+--------+----------------------------------------+
| 14004  | No results.                            |
+--------+----------------------------------------+
