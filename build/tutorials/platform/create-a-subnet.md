# 创建子网

## 简介

[子网](../../../learn/platform-overview/#subnets) 是一组验证程序。一个子网验证一组区块链。每个区块链仅由一个子网进行验证，该子网在区块链创建时就已指定。子网是一种允许创建获许可区块链的强大原语。

创建子网时，会指定阈值和一组密钥。\(实际上，指定的是密钥的地址，而不是密钥本身。\) 为了在子网中添加一个验证程序，需要这些密钥的 _threshold_ 签名。我们将它们称为子网的 **控制密钥** and we call a control key’s signature on a transaction that adds a validator to a subnet a **控制签名** The upshot is that a subnet has control over its membership.

In this tutorial, we’ll create a new subnet with 2 control keys and a threshold of 2.

### Generate the Control Keys <a id="generate-the-control-keys"></a>

First, let’s generate the 2 control keys. To do so we call [`platform.createAddress`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-createaddress) This generates a new private key and stores it for a user.

To generate the first key:

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

This gives the first control key \(again, it actually gives the _address_ of the first control key\). The key is held by the user we just specified.

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-avax1spnextuw2kfzeucj0haf0e4e08jd4499gn0zwg"
    },
    "id": 1
}
```

Generate the second key:

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

The response contains the second control key, which is held by the user we just specified:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-avax1zg5uhuwfrf5tv852zazmvm9cqratre588qm24z"
    },
    "id": 1
}
```

### Create the Subnet <a id="create-the-subnet"></a>

To create a subnet, we call [`platform.createSubnet`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-createsubnet).

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

The response gives us the transaction’s ID, which is also the ID of the newly created Subnet.

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

### Verifying Success <a id="verifying-success"></a>

We can call [`platform.getSubnets`](../../avalanchego-apis/platform-chain-p-chain-api.md#platform-getsubnets) to get all Subnets that exist:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getSubnets",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response confirms that our subnet was created:

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

### Add Validators to the Subnet <a id="add-validators-to-the-subnet"></a>

This [tutorial](../nodes-and-staking/add-a-validator.md) will show you how to add validators to a subnet.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0MDc2MzEwNF19
-->