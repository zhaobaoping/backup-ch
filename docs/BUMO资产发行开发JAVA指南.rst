BUMO资产发行开发JAVA指南
=======================

场景描述
--------

某资方在buchain上发行资产代码为GLA、名称为Global、总发行量为10亿的数字资产，具体信息如下：

+-------------------------+----------+------------------+---------------+
| 字段                    | 是否必填 | 示例             |     描述      |
+=========================+==========+==================+===============+
| name                    | 是       | Global           | 资产名称      |
+-------------------------+----------+------------------+---------------+
| code                    | 是       | GLA              | 资产代码      |
+-------------------------+----------+------------------+---------------+
| totalSupply             | 是       | 1000000000       | 资产总发行量  |
+-------------------------+----------+------------------+---------------+
| decimals                | 是       | 8                | 资产精度      |
+-------------------------+----------+------------------+---------------+
| description             | 否       |                  | 资产描述      |
+-------------------------+----------+------------------+---------------+
| icon                    | 否       |                  | 资产ICON      |
+-------------------------+----------+------------------+---------------+   
| version                 | 是       | 1.0              | 协议版本号    |
+-------------------------+----------+------------------+---------------+ 

.. note:: - code： 推荐使用大写简拼。

       - decimals： 小数位在0~8的范围，0表示无小数位。

       - totalSupply： 范围是0~2^63-1。0表示不固定Token的上限。 
       
       - icon：  base64位编码，图标文件大小是32k以内,推荐200*200像素。

       - version： 协议版本，目前填写1.0。
        



资产发行开发流程
---------------

本文以Java语言为例，新创建一个资产发行方（简称为“资方”）并发行10亿GLA资产。

.. note:: 示例中的[资方账户地址]请替换为资方待发行资产的账户地址，[资方账户私钥]请替换为资方待发行资产的账户私钥。


创建SDK实例
~~~~~~~~~~~

创建实例并设置url(部署的某节点的IP和端口)。

::

 String url = "http://seed1.bumotest.io:26002";
 SDK sdk = SDK.getInstance(url);

在BuChain网络里，每个区块产生时间是10秒，每个交易只需要一次确认即可得到交易终态。

环境说明如下：

+-------------------------+--------------------+------------------+----------------------------------+
| 网络环境                | IP                 | Port             | 区块链浏览器                     |
+=========================+====================+==================+==================================+
| 主网                    | seed1.bumo.io      | 16002            | https://explorer.bumo.io         |
+-------------------------+--------------------+------------------+----------------------------------+
| 测试                    | seed1.bumotest.io  | 26002            | http://explorer.bumotest.io      |
+-------------------------+--------------------+------------------+----------------------------------+


创建资方账户
~~~~~~~~~~~~

创建资方账户的具体代码示例如下：

::

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
     address:  buQYszjqVYdhcPT56GZcKHVh4i7xtx6amr2g
     privateKey:  privbUAYxPLLyaxvU3EMkSTfuEDTWxAYvyCasUcCgUxDihtNXQL4oHJx
     publicKey: b001724ed9475ca4c8893329924c7dceae66c61d8577ab2c2c3b29376e143137c20a4bbed176


.. note:: 通过以上方式创建的账户是未被激活的账户。


激活资方账户
~~~~~~~~~~~~

账户未被激活时需要通过已被激活（已上链）的账户进行激活。已被激活的资方账户请跳过本节内容。


.. note:: - 主网环境：账户激活可以通过小布口袋（钱包）给该资方账户转50.03BU（用于支付资产发行时需要的交易费用），即可激活该账户。

       - 测试环境：资方向 gavin@bumo.io 发出申请，申请内容是资产的账户地址。



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

获取资方账号序列号的返回值如下：

::

 nonce: 28


组装发行资产操作
~~~~~~~~~~~~~~~

一个交易可由多个操作组成，每个操作都指向一个具体的交易内容。
发行资产则需要两个操作: 资产发行操作（AssetIssueOperation）和设置资产信息操作（AccountSetMetadataOperation）。

组装发行资产操作的具体代码如下：

::

    public BaseOperation[] buildOperations() {
    // The account address to issue apt1.0 token
    String issuerAddress = [资方账户地址];
    // The token name
    String name = "Global";
    // The token code
    String code = "GLA";
    // The apt token version
    String version = "1.0";
    // The apt token icon
    String icon = "";
    // The token total supply number
    Long totalSupply = 1000000000L;
    // The token now supply number
    Long nowSupply = 1000000000L;
    // The token description
    String description = "GLA TOKEN";
    // The token decimals
    Integer decimals = 0;

    // Build asset issuance operation
    AssetIssueOperation assetIssueOperation = new AssetIssueOperation();
    assetIssueOperation.setSourceAddress(issuerAddress);
    assetIssueOperation.setCode(code);
    assetIssueOperation.setAmount(nowSupply);

    // If this is an atp 1.0 token, you must set metadata like this
    JSONObject atp10Json = new JSONObject();
    atp10Json.put("name", name);
    atp10Json.put("code", code);
    atp10Json.put("description", description);
    atp10Json.put("decimals", decimals);
    atp10Json.put("totalSupply", totalSupply);
    atp10Json.put("icon", icon);
    atp10Json.put("version", version);

    String key = "asset_property_" + code;
    String value = atp10Json.toJSONString();
    // Build setMetadata
    AccountSetMetadataOperation accountSetMetadataOperation = new AccountSetMetadataOperation();
    accountSetMetadataOperation.setSourceAddress(issuerAddress);
    accountSetMetadataOperation.setKey(key);
    accountSetMetadataOperation.setValue(value);

    BaseOperation[] operations = {assetIssueOperation, accountSetMetadataOperation};
    return operations;
    }

序列化交易
~~~~~~~~~~~~~~~~~~~~~~~~~

序列化交易以便网络传输。


.. note:: - feeLimit: 本次交易发起方最多支付本次交易的交易费用，发行资产操作请填写50.03BU

       - nonce: 本次交易发起方的交易序列号，该值由当前账户的nonce值加1得到。



序列化交易的具体代码如下,示例中的参数nonce是调用getAccountNonce得到的账户序列号，参数operations是调用buildOperations得到发行资产的操作。

::

 public String seralizeTransaction(Long nonce,  BaseOperation[] operations) {
 String transactionBlob = null;

 // The account address to issue atp1.0 token
 String senderAddresss = [资方账户地址];
    // The gasPrice is fixed at 1000L, the unit is MO
    Long gasPrice = 1000L;
    // Set up the maximum cost 50.03BU
    Long feeLimit = ToBaseUnit.BU2MO("50.03");
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
  0A2462755173757248314D34726A4C6B666A7A6B7852394B584A366A537532723978424E4577101C18C0F1CED
  11220E8073A350802122462755173757248314D34726A4C6B666A7A6B7852394B584A366A537532723978424E
  45772A0B0A03474C41108094EBDC033AB6010804122462755173757248314D34726A4C6B666A7A6B7852394B5
  84A366A537532723978424E45773A8B010A1261737365745F70726F70657274795F474C4112757B22636F6465
  223A22474C41222C22746F74616C537570706C79223A313030303030303030302C22646563696D616C73223A3
  02C226E616D65223A22474C41222C2269636F6E223A22222C226465736372697074696F6E223A22474C412054
  4F4B454E222C2276657273696F6E223A22312E30227D

签名交易
~~~~~~~~

所有的交易都需要经过签名后，才是有效的。签名结果包括签名数据和公钥。

签名交易的具体代码如下,示例中的参数transactionBlob是调用seralizeTransaction得到的序列化交易字符串。

::

 public Signature[] signTransaction(String transactionBlob) {
    Signature[] signatures = null;
    // The account private key to issue atp1.0 token
  String senderPrivateKey = [资方账户私钥];


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

 signData: 6CEA42B11253BD49E7F1A0A90EB16448C6BC35E8684588DAB8C5D77B5E771BD5C7E1718942B32F9BDE14551866C00FEBA832D92F88755226434413F98E5A990C; 
 publicKey: b00179b4adb1d3188aa1b98d6977a837bd4afdbb4813ac65472074fe3a491979bf256ba63895


发送交易
~~~~~~~~

将序列化的交易和签名发送到BuChain。

发送交易具体代码如下,示例中的参数transactionBlob是调用seralizeTransaction得到的序列化交易字符串，signatures是调用signTransaction得到的签名数据。

::

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

 hash:  031fa9a7da6cf8777cdd55df782713d4d05e2465146a697832011b058c0a0cd8


查询交易是否执行成功
~~~~~~~~~~~~~~~~~~

.. note:: 发送交易返回的结果只是交易是否提交成功的结果，而交易是否执行成功的结果需要执行如下查询操作, 具体有两种方法：


区块链浏览器查询
^^^^^^^^^^^^^^^

在BUMO区块链浏览器中查询上面的hash，主网(https://explorer.bumo.io)，测试网(http://explorer.bumotest.io)，操作如下图：

|BUBrowser|

查询结果如下：


|execution_result_of_transaction|


调用接口查询
^^^^^^^^^^^^

调用接口查询的代码如下,示例中的参数txHash是调用submitTransaction得到的交易哈希(交易的惟一标识)。

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




.. |BUBrowser| image:: /docs/image/BUBrowser.png
.. |execution_result_of_transaction| image:: /docs/image/execution_result_of_transaction.png

