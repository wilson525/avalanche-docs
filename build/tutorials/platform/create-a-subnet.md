# 创建子网

## 简介

[子网](../../../learn/platform-overview/#subnets) 是一组验证程序。一个子网验证一组区块链。每个区块链仅由一个子网进行验证，该子网在区块链创建时就已指定。子网是一种允许创建获许可区块链的强大原语。

创建子网时，会指定阈值和一组密钥。\(实际上，指定的是密钥的地址，而不是密钥本身。\) 为了在子网中添加一个验证程序，需要这些密钥的 _threshold_ 签名。我们将它们称为子网的 **控制密钥** ，将向子网添加验证程序的交易中**控制密钥** 的签名称为 **控制签名** 。结果就是子网可以控制它的成员。

在本教程中，我们将创建一个具有2个控制密钥和阈值为2的新子网。

### 生成控制密钥<a id="generate-the-control-keys"></a>

首先，让我们生成2个控制密钥。为此，我们需要调用 [`platform.createAddress`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-createaddress) 。这将为客户生成并存储一个新的私人密钥。

生成第一个密钥：

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAddress",
    "params": {
        "username":"USERNAME GOES HERE",
        "password":"PASSWORD GOES HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

这样就生成了第一个控制密钥\(再次声明，它生成的实际上是第一个控制密钥的 _address_ \)。密钥由我们刚刚指定的用户持有。

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-avax1spnextuw2kfzeucj0haf0e4e08jd4499gn0zwg"
    },
    "id": 1
}
```

生成第二个密钥：

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAddress",
    "params": {
        "username":"USERNAME GOES HERE",
        "password":"PASSWORD GOES HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应包含由我们刚刚指定的用户持有的第二个控制密钥：

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-avax1zg5uhuwfrf5tv852zazmvm9cqratre588qm24z"
    },
    "id": 1
}
```

### 创建子网 <a id="create-the-subnet"></a>

要创建子网，我们需要调用[`platform.createSubnet`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-createsubnet).

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createSubnet",
    "params": {
        "controlKeys":[
            "P-avax1spnextuw2kfzeucj0haf0e4e08jd4499gn0zwg",
            "P-avax1zg5uhuwfrf5tv852zazmvm9cqratre588qm24z"
        ],
        "threshold":2,
        "username":"USERNAME GOES HERE",
        "password":"PASSWORD GOES HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

T响应给出了交易的地址，也就是新创建的子网的地址。
```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "3fbrm3z38NoDB4yMC3hg5pRvc72XqnAGiu7NgaEp1dwZ8AD9g",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

### 验证成功 <a id="verifying-success"></a>

可以调用 [`platform.getSubnets`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-getsubnets) 来获取所有存在的子网：

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getSubnets",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应会确认我们的子网已创建：
```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "subnets": [
            {
                "id": "3fbrm3z38NoDB4yMC3hg5pRvc72XqnAGiu7NgaEp1dwZ8AD9g",
                "controlKeys": [
                    "KNjXsaA1sZsaKCD1cd85YXauDuxshTes2",
                    "Aiz4eEt5xv9t4NCnAWaQJFNz5ABqLtJkR"
                ],
                "threshold": "2"
            }
        ]
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

###向子网添加验证程序 <a id="add-validators-to-the-subnet"></a>

本 [教程](../nodes-and-staking/add-a-validator.md) 将向您展示如何向子网添加验证程序。

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MjE5NTA5NDFdfQ==
-->