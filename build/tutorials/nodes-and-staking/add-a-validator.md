# 添加验证程序

## 简介

[主网](https://avalanche.gitbook.io/avalanche/build/tutorials/platform/add-a-validator#introduction) 是Avalanche平台固有的，并验证Avalanche的 [内置区块链](https://avalanche.gitbook.io/avalanche/learn/platform-overview)。本教程中，我们将向Avalanche主网和 [子网](https://avalanche.gitbook.io/avalanche/learn/platform-overview#subnets) 上添加一个节点

P链管理Avalanche上的元数据。其中包括跟踪哪些节点位于哪些子网中、存在哪些区块链以及哪些子网正在验证哪些区块链。如需添加一个验证程序，我们将向P链发起 [交易](http://support.avalabs.org/en/articles/4587384-what-is-a-transaction)

{% hint style="danger" %}
请注意，一旦您发起交易来将一个节点作为验证程序加入时，参数是无法更改的。**您无法提前移除质押或更改质押金额、节点ID或收益地址。** 请确保您在以下API调用中使用了正确的值。如果您不确定，请浏览 [开发人员FAQ](http://support.avalabs.org/en/collections/2618154-developer-faq) 或在 [Discord.](https://chat.avalabs.org/)上寻求帮助。
{% endhint %}

## 要求

您已完成 [行一个Avalanche节点](../../getting-started.md) ，并了解 [Avalanche架构](../../../learn/platform-overview/)。本教程中，我们使用 [Avalanche Postman收藏夹](https://github.com/ava-labs/avalanche-postman-collection) 来帮助我们进行API调用。

为了确保您连好了您的节点，请确保您的节点能够在锁定端口上收到并发送TCP流量 \(`9651` by default\) ，并且您使用命令行参数启动了节点 `--public-ip=[您的节点的公共IP]`。如果未做到这一点，则您的质押收益可能会受到影响。

## 向Avalanche钱包添加验证程序

首先，我们向您展示如何使用 [Avalanche 钱包](https://wallet.avax.network)将您的节点作为验证程序添加进来

如需获取您的节点ID，请调用 [`info.getNodeID`](https://avalanche.gitbook.io/avalanche/build/apis/info-api#info-getnodeid):

![获取节点ID postman](../../../.gitbook/assets/getNodeID-postman.png)

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

响应中包含您的节点ID：

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD"
    },
    "id": 1
}
```

打开 [钱包](https://wallet.avax.network/)，并进入`Earn` 选项。选择 `Add Validator`。

![线上钱包收入选项](../../../.gitbook/assets/web-wallet-earn-tab.png)

填写质押参数。下面将进行更详细的讲解。当您填写完所有质押参数并复查时，请点击`Confirm`请确认质押期至少是2周，委托费率至少是2%，并且您至少质押了2000AVAX。

{% page-ref page="../../../learn/platform-overview/staking.md" %}

![获得验证](../../../.gitbook/assets/earn-validate.png)

您应该查看交易成功的信息，并且您的余额也应更新。

![您的验证交易已发送](../../../.gitbook/assets/your-validation-transaction-is-sent.png)

调用 [`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) 核实了我们的交易已接受。

![获取挂起验证程序 postman](../../../.gitbook/assets/getPendingValidators-postman.png)

返回 `Earn` 选项，并点击 `Estimated Rewards`.

![赚钱、验证、授权](../../../.gitbook/assets/earn-validate-delegate.png)

一旦您的验证程序开始时间过了，您就会看到它可能赚取的收益、开始时间、结束时间机器验证期已经过去的百分比。

![预计收益](../../../.gitbook/assets/estimated-rewards.png)

就是这样！

## 向API调用添加验证程序

通过向我们的节点进行API调用，您可以向验证程序集合添加一个节点。要将节点添加到主网，我们将调用[`platform.addValidator`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-addvalidator).

此方法的明显特征是：

```cpp
platform.addValidator(
    {
        nodeID: string,
        startTime: int,
        endTime: int,
        stakeAmount: int,
        rewardAddress: string,
        changeAddr: string, (optional)
        delegationFeeRate: float,
        username: string,
        password: string
    }
) -> {txID: string}
```

让我们看看这些论点。

`nodeID`

这是将要添加的验证程序的节点ID。如需获取您的节点ID，请调用 [`info.getNodeID`](https://avalanche.gitbook.io/avalanche/build/apis/info-api#info-getnodeid):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "info.getNodeID",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

响应中包含您的节点ID：

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji"
    },
    "id": 1
}
```

`开始时间` 与 `结束时间`

当一个人发起一项交易，并加入主网时，即确定了输入 \(开始验证\) 和离开 \(结束验证\) 的时间。验证主网的最短持续时间为24小时，最长持续时间为一年。离开后可以重新进入主网，但最长持续时间是一年。 `开始时间` and `结束时间` 分别是您的验证程序启动和停止验证主网的Unix次数。`开始时间` 必须是在交易发起时间之后。
`stakeAmount`

为了验证主网，您必须质押AVAX。该参数确定了质押的AVAX金额。

`rewardAddress`

当验证程序停止验证主网时，如果它们验证主网时响应足够且正确，则它们会获得收益。上述代币会发送到`rewardAddress`。原始质押会发送回 `username`控制的一个地址。

无论行为如何，验证程序的质押永远不会大幅削减；完成验证时，它们会一直收回其质押。

`changeAddr`

任何关于此交易的更改都会发送到此地址。您可以留空此区域；如果您留空了，更改会发送到您的用户名下的一个地址。

`delegationFeeRate`

通过Avalanche，可以进行质押委托。该参数是其他人将股份委托给验证程序时，验证程序收取的百分比费用。例如，如果`delegationFeeRate` 为 `1.2345` ，并且有人向验证程序提出委托时，则委托期结束时，1.2345%的收益会分配给验证程序，其余部分则会分配给委托方。

`username` 与 `password`

这些参数是支付交易费用、提供质押AVAX和回收质押AVAX的用户的用户名与密码。

现在让我们发起交易吧。我们使用外壳命令 `date` 来计算Unix次数，将来，10分钟和30天会分别被用作 `startTime` 与 `endTime`的值。 \(注意事项：如果您使用的是Mac，请用 `$(date` 来替换 `$(gdate`。如果您并未安装 `gdate` ，请执行`brew install coreutils`.\) 本例中，我们质押了2,000 AVAX \(2 x 1012 nAVAX\).

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "stakeAmount":2000000000000,
        "rewardAddress":"P-avax1d4wfwrfgu4dkkyq7dlhx0lt69y2hjkjeejnhca",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "delegationFeeRate":10,
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应包含交易ID和收到更改的地址。

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

我们可以通过调用[`platform.getTxStatus`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-gettxstatus)来查看交易状态:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

状态应是`Committed`，意即交易成功。我们可以调用[`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) ，并查看该节点目前正处于主网的挂起验证程序集合中：

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应包括我们刚刚添加的节点：

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime": "1584021450",
                "endtime": "1584121156",
                "stakeAmount": "2000000000000",
            }
        ] 
    },
    "id": 1
}
```

当时间达到 `1584021450`时，该节点将开始验证主网。当时间达到`1584121156`，该节点将停止验证主网。质押的AVAX会退还至`username`控制的地址，如有，则收益会发送到 `rewardAddress`.

## 添加子网验证程序

### 发起子网验证程序交易

现在让我们把同一个节点添加到子网中。如果您已经完成了 [创建子网教程](https://avalanche.gitbook.io/avalanche/build/tutorials/platform/create-a-subnet)，那么下文会更有意义。现在，您只能通过API调用将验证程序添加到子网中，通过Avalanche钱包是行不通的。

假如子网的ID是`nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`，阈值是2，则 `username` 至少持有2个控制密钥。

如需添加验证程序，我们将调用API法[`platform.addSubnetValidator`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-addsubnetvalidator)。其特征是：

```cpp
platform.addSubnetValidator(
    {
        nodeID: string,
        subnetID: string,
        startTime: int,
        endTime: int,
        weight: int,
        changeAddr: string, (optional)
        username: string,
        password: string
    }
) -> {txID: string}
```

让我们来检查一下参数：

`nodeID`

这是添加到子网的验证程序的节点ID。**此验证程序必须在其验证此子网的全程中验证主网。**

`subnetID`

这是我们要将验证程序添加进去的子网的ID。

`startTime` 与 `endTime`

与上文类似，这是验证程序将开始和停止验证子网的Unix次数。`startTime` 必须在验证程序开始验证主网的同时或之后，而 `endTime` 必须在验证程序停止验证主网的同时或之前。

`weight`

这是为达成共识的验证程序的采样权重。如果验证程序的权重是1，且子网中所有验证程序的累计权重是100，则达成共识期间，每100个样本中就有1个包含该验证程序。

`changeAddr`

关于此交易的任何更改都会发送到此地址。您可以留空此区域；如果您留空了，更改会发送到您的用户控制的一个地址。

`username` 与 `password`

这些参数是支付交易费用的用户的用户名与密码。该用户持有足量子网的控制密钥，进而将验证程序添加到此子网中。

我们使用外壳命令 `date` 来计算Unix次数，将来，10分钟和30天会分别被用作 `startTime` 和`endTime`的值。\(注意事项：如果您使用的是Mac，请用`$(date` 来替换 `$(gdate`如果您并未安装`gdate` ，请执行 `brew install coreutils`.\)

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addSubnetValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "weight":1,
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应包含交易ID和收到更改的地址。

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

我们可以通过调用[`platform.getTxStatus`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-gettxstatus)来查看交易状态:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

状态应是`Committed`，意即交易成功。我们可以调用[`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) ，并查看该节点目前正处于主网的挂起验证程序集合中。此次，我们制定了子网ID：

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {"subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr"},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

响应包括我们刚刚添加的节点：

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime":1584042912,
                "endTime":1584121156,
                "weight": "1"
            }
        ]
    },
    "id": 1
}
```

当时间达到 `1584042912`时，该节点将开始验证主网。当时间达到`1584121156`，该节点将停止验证主网。

### 将子网纳入白名单

目前该节点已经添加为子网的验证程序，让我们把它添加到子网白名单中。白名单防止节点无意中验证子网。

To whitelist the subnet, restart the node and add the parameter `--whitelisted-subnets` with a comma separated list of subnets to whitelist.

The full command is:

`./build/avalanchego --whitelisted-subnets=nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQzNjc5OTU2OSwtMTk5OTM0MTc3Nl19
-->