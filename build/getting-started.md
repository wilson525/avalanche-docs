# 准备开始：运行Avalanche节点

想要了解Avalanche，最快的方式是运行一个节点并进行网络互动。

{% embed url="https://youtu.be/c\_SjtCiOFdg" %}

本教程中\(预计时间：10 分钟\)，我们将：

* 安装并运行一个Avalanche节点
* 连接到Avalanche
* 发送AVAX
* 将您的节点添加到验证程序集合中

如果您的问题在FAQ中未能解决，请在[Avalanche Discord](https://chat.avax.network)中寻求帮助！我们会帮助您解决任何困难。

本教程主要面向开发人员和对Avalanche平台如何运行感兴趣的人士。如果您只是对设置质押节点感兴趣，您可以查询 [通过安装程序设置Avalanche节点](tutorials/nodes-and-staking/set-up-node-with-installer.md) 教程。安装程序自动化安装过程并将其设置为一项系统服务，该安装方式推荐用于无人值守操作。您也可以先根据本教程进行尝试，然后永久使用安装程序来设置节点。

## 要求

Avalanche是一项非常轻量级的协议，因此对计算机的最低要求不高。

* 硬件：CPU > 2 GHz，内存> 4 GB，存储> 10 GB免费空间
* 操作系统：Ubuntu 18.04/20.04或MacOS >= Catalina

## 运行Avalanche节点和发送资金
安装AvalancheGo（即Avalanche节点的GO安装启用），并连接到Avalanche公共测试网。

### 下载AvalancheGo
该节点是一个二进制程序。您可以在下载源代码后构建二进制程序，也可以下载预构建的二进制程序。二者选其一即可。

如果您只是想要运行您自己的节点并对其进行质押，则更推荐下载 [预构建的二进制程序](getting-started.md#binary) ，该方式更为简单。

如果您是一名想在在Avalanche上进行实验和构建的开发人员，则推荐根据源代码构建节点。

#### **源代码**

如果您想根据源代码构建节点，您需要先安装Go 1.15.5或更新版本。请遵循 [此处](https://golang.org/doc/install) 指示

运行 `go version`. **版本应是1.15.5或更新版本** 运行 `echo $GOPATH`. **不应空白。**

下载AvalancheGo存储库：

```cpp
go get -v -d github.com/ava-labs/avalanchego/...
```

高级用户注意事项：AvalancheGo使用Go模块，因此您可以将 [AvalancheGo 存储库](https://github.com/ava-labs/avalanchego) 复制到GOPATH以外的其他位置。

切换到 `avalanchego` 目录：

```cpp
cd $GOPATH/src/github.com/ava-labs/avalanchego
```

构建AvalancheGo：

```cpp
./scripts/build.sh
```

`avalanchego`二进制程序在`avalanchego/build`目录下。

#### **二进制**

如果您想要下载一个预构建的二进制程序，而不是自己构建，请访问我们的 [版本页面](https://github.com/ava-labs/avalanchego/releases)，并选择您需要的版本\(可能是最新版本\)

`资产`项下，请选择适当的文件。

如果是Mac操作系统：
下载： `avalanchego-macos-<VERSION>.zip`  
解压： `unzip avalanchego-macos-<VERSION>.zip`  
生成的文件夹 `avalanchego-<VERSION>`包含二进制程序。

如果是个人电脑或云供应商的Linux系统：
下载： `avalanchego-linux-amd64-<VERSION>.tar.gz`  
解压： `tar -xvf avalanchego-linux-amd64-<VERSION>.tar.gz`  
生成的文件夹 `avalanchego-<VERSION>-linux`包含二进制程序。

如果是树莓派4或类似Arm64系统电脑的Linux系统：
下载： `avalanchego-linux-arm64-<VERSION>.tar.gz`  
解压： `tar -xvf avalanchego-linux-arm64-<VERSION>.tar.gz`  
生成的文件夹 `avalanchego-<VERSION>-linux`包含二进制程序。

### 启动一个节点，并连接到Avalanche

如果您根据源代码进行构建：

```cpp
./build/avalanchego
```

如果您在Mac操作系统上使用预构建的二进制程序：

```cpp
./avalanchego-<VERSION>/build/avalanchego
```

如果您在Linux系统上使用预构建的二进制程序：

```cpp
./avalanchego-<VERSION>-linux/avalanchego
```

节点启动时，它必须进行引导\(与网络的其他部分保持同步\)。您会看到有关引导的日志。当一个给定链完成了引导，则会打印出如下一份日志：
`INFO [06-07|19:54:06] <X Chain> /snow/engine/avalanche/transitive.go#80: bootstrapping finished with 1 vertices in the accepted frontier`

如果一个给定链完成了引导，则通过复制粘贴以下命令签入另一个终端窗口调用 [`info.isBootstrapped`](avalanchego-apis/info-api.md#info-isbootstrapped)

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.isBootstrapped",
    "params": {
        "chain":"X"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

如果反馈`true`，则链完成引导。如果您向一个未完成引导的链进行API调用，则该链会反馈 `API call rejected because chain is not done bootstrapping`。如果您的节点并未完成引导，请遵循 [this FAQ](http://support.avalabs.org/en/articles/4593908-is-my-node-done-bootstrapping)，如果您还有问题，请通过 [Discord.](https://chat.avalabs.org/)联系我们。

当前您的节点正在运行且已连接。如果您想在主网上将您的节点用作验证程序，请查看 [教程](tutorials/nodes-and-staking/add-a-validator.md#add-a-validator-with-avalanche-wallet) ，以了解如何使用web钱包将节点添加为验证程序。
您可以按下 `Ctrl + C` 来终止该节点。

如果您想要实验并使用您的节点，请继续阅读。
为了能够从其他计算机对您的节点进行API调用，在启动节点时，请包含参数 `--http-host=` \(例如 `./build/avalanchego --http-host=`\)

如果要连接到富士测试网，而不是主网，请使用参数 `--network-id=fuji`。您可以通过 [faucet.](https://faucet.avax-test.network/)在测试网上获得资金。

### 创建密钥库用户

Avalanche节点提供了一个内置 **密钥库.** 。密钥库对用户进行管理，而且很像一个 [钱包](http://support.avalabs.org/en/articles/4587108-what-is-a-blockchain-wallet)。用户是一个受密码保护的身份，客户端与区块链交互时可以使用该身份。 **由于节点操作人员能够访问您的明文密码，因此您只能在您操作的节点上创建密钥库用户。** 如果要创建用户，请调用 [`keystore.createUser`](avalanchego-apis/keystore-api.md#keystore-createuser):

```cpp
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "YOUR USERNAME HERE",
         "password": "YOUR PASSWORD HERE"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

响应应当是：

```cpp
{
     "jsonrpc":"2.0",
     "result":{"success":true},
     "id":1
}
```

目前，该节点上您有一个用户账号。节点层面上存在密钥库数据。您在一个节点上密钥库里创建的用户在其他节点上不存在，但是您可以将用户导入/导出到密钥库。如需了解如何使用，请查阅 [密钥库 API](avalanchego-apis/keystore-api.md)。

{% hint style="danger" %}
**您应在您的节点上只保留少量资金。** 您的大部分资金应由一个不会保存到任何计算机的助记符号进行保护。
{% endhint %}

### 创建地址

Avalanche是一个异构区块链平台， [X-Chain](../learn/platform-overview/#exchange-chain-x-chain)是其中一种，其作用是创建和交易数字资产的去中心化平台。目前，我们要创建地址来保存节点上的AVAX。

如果要在X链上创建一个新地址，请调用[`avm.createAddress`](avalanchego-apis/exchange-chain-x-chain-api.md#avm-createaddress),  [X-Chain’s API](avalanchego-apis/exchange-chain-x-chain-api.md)方法：

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :2,
    "method" :"avm.createAddress",
    "params" :{
        "username":"YOUR USERNAME HERE",
        "password":"YOUR PASSWORD HERE"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

如果您的节点未完成引导，则此次调用会返回`503` 状态，并附信息 `API call rejected because chain is not done bootstrapping`。

请注意，我们向`127.0.0.1:9650/ext/bc/X`发出此请求。`bc/X` 部分表明了请求正在发送至ID \(或别名\)为`X` \(例如X链\).

响应应如下：

```cpp
{
    "jsonrpc":"2.0",
    "id":2,
    "result" :{
        "address":"X-avax1xeaj0h9uy7c5jn6fxjp0rg4g39jeh0hl27vf75"
    }
}
```

目前您的用户可以控制X链上的地址 `X-avax1xeaj0h9uy7c5jn6fxjp0rg4g39jeh0hl27vf75` 。为了区分不同链上的地址，Avalanche约定是用于包含其所在链的ID或别名的一个地址。因此，地址的开头是 `X-`，说明其存在于X链上。

### 从Avalanche钱包向您的节点发送资金

{% hint style="warning" %}
_**注意事项：转移实际资金的指南如下。**_
{% endhint %}

来把Avalanche钱包中的资金转到您的节点上吧。

访问 [Avalanche Wallet](https://wallet.avax.network). 。点击 `Access Wallet`，再点击 `Mnemonic Key Phrase` 输入您的助记词。

点击左侧的 `Send` 键。金额选择 `.002` AVAX。输入您的节点地址，然后点击 `Confirm`.

![web wallet send tab](../.gitbook/assets/web-wallet-send-tab%20%284%29%20%284%29%20%285%29%20%285%29%20%286%29%20%287%29%20%283%29.png)

通过调用 `avm.getBalance`，另一种X链API方法，我们可以检查地址的给定资产余额。检查一下转账是否成功：

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :3,
    "method" :"avm.getBalance",
    "params" :{
        "address":"X-avax1xeaj0h9uy7c5jn6fxjp0rg4g39jeh0hl27vf75",
        "assetID"  :"AVAX"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

请注意，AVAX有一个特殊ID `AVAX`。通常来说，一个资产ID是一个字母数字串。
响应应标明我们拥有`2,000,000 nAVAX` 或 `0.002 AVAX`.

```cpp
{
    "jsonrpc":"2.0",
    "id"     :3,
    "result" :{
        "balance":2000000,
        "utxoIDs": [
            {
                "txID": "x6vR85YPNRf5phpLAEC7Sd6Tq2PXWRt3AAHAK4BpjxyjRyhtu",
                "outputIndex": 0
            }
        ]
    }
}
```

### 发送AVAX

现在，让我们向我们的节点进行一次API调用，然后发送一些AVAX吧。

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :5,
    "method" :"avm.send",
    "params" :{
        "assetID"    :"AVAX",
        "amount"     :1000,
        "to"         :"X-avax1w4nt49gyv4e99ldqevy50l2kz55y9efghep0cs",
        "changeAddr" :"X-avax1turszjwn05lflpewurw96rfrd3h6x8flgs5uf8",
        "username"   :"YOUR USERNAME HERE",
        "password"   :"YOUR PASSWORD HERE"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

`amount` 规定了需要发送的nAVAX数量。

如果您想要指定一个特定地址，您可以在 `changeAddr`中进行指定。您可空白此字段；如果您空白了，则如有任何更改，均会显示在您的用户账号控制下的其中一个地址。

为了拦截垃圾邮件，Avalanche要求支付交易费用。如果您发起一项交易，则交易费用会从您的用户账号控制下的一个地址自动扣除。请您在检查下列余额时记住这一点。

{% page-ref page="../learn/platform-overview/transaction-fees.md" %}

当您发送此请求时，该节点会通过您的用户名和密码核实您的身份。然后，该节点会检查您的用户账号控制下的所有 [私钥](http://support.avalabs.org/en/articles/4587058-what-are-public-and-private-keys) ，直至该节点找到足够的AVAX来满足此请求。

响应包含交易ID。每次调用 `send` 都会有所不同。

```cpp
{
    "jsonrpc":"2.0",
    "id"     :5,
    "result" :{
        "txID":"2QouvFWUbjuySRxeX5xMbNCuAaKWfbk5FeEa2JmoF85RKLk2dD",
        "changeAddr" :"X-avax1turszjwn05lflpewurw96rfrd3h6x8flgs5uf8"
    }
}
```

####  检查交易状态

该交易仅需一两秒即可完成。我们可以使用[`avm.getTxStatus`](avalanchego-apis/exchange-chain-x-chain-api.md#avm-gettxstatus)来检查其状态：

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :6,
    "method" :"avm.getTxStatus",
    "params" :{
        "txID":"2QouvFWUbjuySRxeX5xMbNCuAaKWfbk5FeEa2JmoF85RKLk2dD"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

响应应表明该交易已接受：

```cpp
{
    "jsonrpc":"2.0",
    "id"     :6,
    "result" :{
        "status":"Accepted"
    }
}
```

如果网络尚未完成交易，您可能会看到 `status` 是`Processing` 

一旦您看到交易 `Accepted`请检查 `to` 地址中的余额是否已收到我们发送的AVAX：

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :7,
    "method" :"avm.getBalance",
    "params" :{
        "address":"X-avax1w4nt49gyv4e99ldqevy50l2kz55y9efghep0cs",
        "assetID"  :"AVAX"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

响应应当是：

```cpp
{
    "jsonrpc":"2.0",
    "id"     :7,
    "result" :{
        "balance":1000
    }
}
```

同样地，我们可以检查 `X-avax1xeaj0h9uy7c5jn6fxjp0rg4g39jeh0hl27vf75` ，以查看我们发送的AVAX和交易费用是否已从其余额中扣除。

{% page-ref page="tutorials/nodes-and-staking/add-a-validator.md" %}

{% page-ref page="tools/avalanchejs/create-an-asset-on-the-x-chain.md" %}

{% page-ref page="tutorials/platform/create-a-new-blockchain.md" %}

{% page-ref page="tutorials/platform/create-a-subnet.md" %}

{% page-ref page="avalanchego-apis/" %}

{% page-ref page="references/" %}

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM3MTMzNjUwLC04NDYxNDc3OTFdfQ==
-->