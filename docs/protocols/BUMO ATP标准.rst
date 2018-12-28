BUMO ATP标准
============

概述
----

ATP1.0（Account-based Tokenization Protocol）指基于 BuChain 的账号结构发行、转移以及增发 token 的标准协议。在本文档中 token 表示账号资产。

目标
--------

通过ATP协议让其他应用程序方便地调用接口，在 BUMO 上发行、转移以及增发 token 等操作。

Token属性参数
-------------

通过设置 token 源账户的 metadata 可以设置已发行 token 的相关属性，方便应用程序管理和查询 token 数据信息。

+--------------+----------------------------------------------+
| 变量         | 描述                                         |
+==============+==============================================+
| name         | token 名称                                   |
+--------------+----------------------------------------------+
| code         | token 代码                                   |
+--------------+----------------------------------------------+
| description  | token 描述                                   |
+--------------+----------------------------------------------+
| decimals     | token 小数位数                               |
+--------------+----------------------------------------------+
| totalSupply  | token 总量（其值等于10 ^ decimals * 发行量） |
+--------------+----------------------------------------------+
| icon         | token 图标（optional）                       |	
+--------------+----------------------------------------------+	
| version      | ATP 版本                                     |
+--------------+----------------------------------------------+ 

.. note:: 

 - code：推荐使用大写简拼。
 - decimals：小数位在 0~8 的范围，0 表示无小数位。
 - totalSupply：范围是 0~2^63-1。0 表示不固定 token 的上限。例如，发行一笔数量是10000、精度为 8 的ATP1.0 token，则 totalSupply = 10 ^ 8 * 10000, 即totalSupply为 1000000000000。
 - icon：base64 位编码，图标文件大小是 32k 以内，推荐 200*200 像素。

操作
--------

BUMO ATP标准中的操作包括 `登记token`_、`发行token`_、`转移token`_、`增发token`_、`查询token`_、`查询指定metadata`_。

登记token
^^^^^^^^^^

登记 token 即设置 token 的 metadata 参数。可通过发送 ``Setting Metadata`` 交易设置 token 的 metadata 参数 key、value 和 version。下面是登记 token 的示例:

**json格式**

::

 {
  "type": 4,
  "set_metadata": {
    "key": "asset_property_DT",
    "value": "{\"name\":\"Demon Token\",\"code\":\"DT\",\"totalSupply\":\"10000000000000\",\"decimals\":8,
    \"description\":\"This is hello Token\",\"icon\":\"iVBORw0KGgoAAAANSUhEUgAAAAE....\",\"version\":\"1.0\"}",
    "version": 0
  }
 }

.. note::

 key 值必须是 asset_property_ 前缀和 token code 的组合（参考发行 token 的 code 参数）。
 设置成功后通过查询指定 metadata 可以看到相关数据。

发行token
^^^^^^^^^^

发行 token 即账户发行一笔数字 token，执行成功后账户的 token 余额中会出现这一笔 token。客户端通过发起一笔操作类型是 ``Issuing Assets`` 的交易，设置参数 amount（发行的数量）、code（token 代码）。
下面是发行一笔数量是 10000，精度为 8 的 DT token 的示例，此时参数 amount 是10 ^ 8 * 10000：

**json格式**

::

 {
  "type": 2,
  "issue_asset": {
    "amount": 1000000000000,
    "code": "DT"
  }
 }

转移token
^^^^^^^^^

转移 token 即账户将一笔 token 转给目标账户。转移 token 通过发送 ``Transferring Assets`` 的交易设置相关参数。以下是相应参数：

+----------------------------------+------------------------------------+
| 参数                             | 描述                               |
+==================================+====================================+
| pay_asset.dest_address           | 目标账户地址                       |
+----------------------------------+------------------------------------+
| pay_asset.asset.key.issuer       | token 发行方地址                   |
+----------------------------------+------------------------------------+
| pay_asset.asset.key.code         | token 代码                         |
+----------------------------------+------------------------------------+
| pay_asset.asset.amount           | 10 ^ 精度 * 要转移的数量           |
+----------------------------------+------------------------------------+
| pay_asset.input                  | 触发合约调用的入参，默认为空字符串 |
+----------------------------------+------------------------------------+


下面是给已激活的目标账户 buQaHVCwXj9ERtFznDnAuaQgXrwj2J7iViVK 转移数量为 500000000000 的 DT 的例子：

**json格式**

::

    {
      "type": 3,
      "pay_asset": {
        "dest_address": "buQaHVCwXj9ERtFznDnAuaQgXrwj2J7iViVK",
        "asset": {
          "key": {
            "issuer": "buQhzVyca8tQhnqKoW5XY1hix2mCt5KTYzcD",
            "code": "DT"
          },
          "amount": 500000000000
        }
      }
    }

转移成功后通过查询 token 可以看到目标账户拥有 amount 数量的 DT。

.. note:: 给未激活的目标账户转移 token，交易执行失败。

增发token
^^^^^^^^^

增发 token 即账户继续在原 token 代码上发行一定数量的 token，通过设置和之前发行的 token 相同的交易类型代码，继续发送发行 token 的交易。
应用程序根据具体业务去控制增发 token 的数量是否超过 totalSupply，增发成功后会看到 token 数量增加。

查询token
^^^^^^^^^^

查询 token 即查询源账户的 token 信息，以下是查询 token 需要指定的 token 信息:

+----------------------------------+---------------------------------------------------+
| 参数                             | 描述                                              |
+==================================+===================================================+
| address                          | 账号地址，必填                                    |
+----------------------------------+---------------------------------------------------+
| code &                           | issuer 表示 token 的发行账户地址，                |
| issuer                           | code 表示 token 代码。只有同时填写正确的          |
|                                  | code&issuer 才能正确显示指定的 token，            |
|                                  | 否则默认显示所有 token。                          |
+----------------------------------+---------------------------------------------------+
| type                             | 目前 type 只能是 0，可以不用填写。                |
+----------------------------------+---------------------------------------------------+

以下是查询 token 的代码示例：


::

 HTTP GET /getAccountAssets?address=buQhzVyca8tQhnqKoW5XY1hix2mCt5KTYzcD




如果该账号存在 token，则返回内容:

::

 
 {
    "error_code": 0,
    "result": [
        {
            "amount": 469999999997,
            "key": {
                "code": "DT",
                "issuer": "buQhzVyca8tQhnqKoW5XY1hix2mCt5KTYzcD"
            }
        },
        {
            "amount": 1000000000000,
            "key": {
                "code": "ABC",
                "issuer": "buQhzVyca8tQhnqKoW5XY1hix2mCt5KTYzcD"
            }
        }
    ]
 }

如果该账号不存在 token，则返回内容:

::

 {
   "error_code" : 0,
   "result" : null
 }

查询指定metadata
^^^^^^^^^^^^^^^^^

查询指定 metadata 即查询 metadata 的相关信息，包括 key、value、version。查询 metadata 需指定的 metadata 信息:

+----------------------------------+---------------------------------------------------+
| 参数                             | 描述                                              |
+==================================+===================================================+
| address                          | 账号地址，必填。                                  |
+----------------------------------+---------------------------------------------------+
| key                              | 指定 metadata 中的 key 值。                       |
+----------------------------------+---------------------------------------------------+

以下是查询指定 metadata 的代码示例：

::

 HTTP GET /getAccountMetaData?address=buQhzVyca8tQhnqKoW5XY1hix2mCt5KTYzcD&key=asset_property_DT


如果该账号指定的 key 存在 metadata，则返回内容:

::

 {
    "error_code": 0,
    "result": {
        "asset_property_DT": {
            "key": "asset_property_DT",
            "value": "{\"name\":\"DemonToken\",\"code\":\"DT\",\"totalSupply\":\"1000000000000\",\"decimals\":8,\"description\":\"This is hello Token\",\"icon\":\"iVBORw0KGgoAAAANSUhEUgAAAAE\",\"version\":\"1.0\"}",
            "version": 4
        }
    }
 }

如果该账号指定的 key 不存在 metadata，则返回内容:

::

 {
   "error_code" : 0,
   "result" : null
 }
