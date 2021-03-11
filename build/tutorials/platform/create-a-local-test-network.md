# 创建本地测试网络

## 简介

在 [入门教程](https://avalanche.gitbook.io/avalanche/build/getting-started)中，我们将一个节点连接到了测试网络。您可能会发现创建一个本地测试网络非常有用。

接下来我们将向您展示如何启动一个5节点的本地测试网络。我们将展示如何使用 [Avash](https://avalanche.gitbook.io/avalanche/build/tools/avash) 启动网络和手动启动网络。

这5个节点将拥有五个HTTP端口（API 调用的发送地址）：`9650`, `9652`, `9654`, `9656` , 和`9658`.

## 创建本地测试网络

下面的命令假设您已经在`$GOPATH/src/github.com/ava-labs/avalanchego`上安装了[AvalancheGo](https://avalanche.gitbook.io/avalanche/build/getting-started#download-avalanchego) 。创建的每一个节点都是一个验证程序。这些节点的质押密钥在`$GOPATH/src/github.com/ava-labs/avalanchego/staking/local/staker1.crt`等

### 手动

启动网络：

```cpp
cd $GOPATH/src/github.com/ava-labs/avalanchego
```

```cpp
./scripts/build.sh
```

```cpp
./build/avalanchego --public-ip=127.0.0.1 --snow-sample-size=2 --snow-quorum-size=2 --http-port=9650 --staking-port=9651 --db-dir=db/node1 --staking-enabled=true --network-id=local --bootstrap-ips= --staking-tls-cert-file=$(pwd)/staking/local/staker1.crt --staking-tls-key-file=$(pwd)/staking/local/staker1.key
```

```cpp
./build/avalanchego --public-ip=127.0.0.1 --snow-sample-size=2 --snow-quorum-size=2 --http-port=9652 --staking-port=9653 --db-dir=db/node2 --staking-enabled=true --network-id=local --bootstrap-ips=127.0.0.1:9651 --bootstrap-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg --staking-tls-cert-file=$(pwd)/staking/local/staker2.crt --staking-tls-key-file=$(pwd)/staking/local/staker2.key
```

```cpp
./build/avalanchego --public-ip=127.0.0.1 --snow-sample-size=2 --snow-quorum-size=2 --http-port=9654 --staking-port=9655 --db-dir=db/node3 --staking-enabled=true --network-id=local --bootstrap-ips=127.0.0.1:9651 --bootstrap-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg --staking-tls-cert-file=$(pwd)/staking/local/staker3.crt --staking-tls-key-file=$(pwd)/staking/local/staker3.key
```

```cpp
./build/avalanchego --public-ip=127.0.0.1 --snow-sample-size=2 --snow-quorum-size=2 --http-port=9656 --staking-port=9657 --db-dir=db/node4 --staking-enabled=true --network-id=local --bootstrap-ips=127.0.0.1:9651 --bootstrap-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg --staking-tls-cert-file=$(pwd)/staking/local/staker4.crt --staking-tls-key-file=$(pwd)/staking/local/staker4.key
```

```cpp
./build/avalanchego --public-ip=127.0.0.1 --snow-sample-size=2 --snow-quorum-size=2 --http-port=9658 --staking-port=9659 --db-dir=db/node5 --staking-enabled=true --network-id=local --bootstrap-ips=127.0.0.1:9651 --bootstrap-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg --staking-tls-cert-file=$(pwd)/staking/local/staker5.crt --staking-tls-key-file=$(pwd)/staking/local/staker5.key
```

### 通过 Avash

假设您已安装 [Avash](https://avalanche.gitbook.io/avalanche/build/tools/avash).

打开 Avash：

```cpp
cd $GOPATH/src/github.com/ava-labs/avash
```

```cpp
go build
```

```cpp
./avash
```

现在我们已经打开Avash。启动网络：

```cpp
runscript scripts/five_node_staking.lua
```

当您想要关闭网络时，运行`exit` 退出 Avash。

### 检查节点是否连接 <a id="verifying-nodes-are-connected"></a>

可以通过检查与一个节点相连的其他节点来确保所有节点都已连接。为此，调用[`info.peers`](https://avalanche.gitbook.io/avalanche/build/apis/info-api#info-peers).

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.peers"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

`peers` 应该有4个条目：
```cpp
{
    "jsonrpc":"2.0",
    "result":{
        "numPeers":"4",
        "peers":[
            {
                "ip":"127.0.0.1:36698",
                "publicIP":"127.0.0.1:9655",
                "nodeID":"NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN",
                "version":"avalanche/1.0.5",
                "lastSent":"2020-11-15T09:29:16-05:00",
                "lastReceived":"2020-11-15T09:29:09-05:00"
            },
            {
                "ip":"127.0.0.1:37036",
                "publicIP":"127.0.0.1:9657",
                "nodeID":"NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu",
                "version":"avalanche/1.0.5",
                "lastSent":"2020-11-15T09:29:16-05:00",
                "lastReceived":"2020-11-15T09:29:18-05:00"
            },
            {
                "ip":"127.0.0.1:38764",
                "publicIP":"127.0.0.1:9659",
                "nodeID":"NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5",
                "version":"avalanche/1.0.5",
                "lastSent":"2020-11-15T09:29:16-05:00",
                "lastReceived":"2020-11-15T09:29:15-05:00"
            },
            {
                "ip":"127.0.0.1:60194",
                "publicIP":"127.0.0.1:9653",
                "nodeID":"NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ",
                "version":"avalanche/1.0.5",
                "lastSent":"2020-11-15T09:29:16-05:00",
                "lastReceived":"2020-11-15T09:29:09-05:00"
            }
        ]
    },
    "id":1
}
```

### 获得 AVAX <a id="getting-avax"></a>

当运行一个 `--network-id=local`的网络时，正如我们所做的那样，您可以导入一个预先投资的X链地址来获得AVAX。该地址的私人密钥为 `PrivateKey-ewoqjP7PxY4yr3iLTpLisriqt94hdyDFNgchSxGGztUrTXtNN`。在节点上创建密钥库用户之后，您可以通过以下方式导入该密钥及其持有的资金：

```cpp
curl --location --request POST 'localhost:9650/ext/platform' \
--header 'Content-Type: application/json' \
--data-raw '{
    "jsonrpc": "2.0",
    "method": "platform.importKey",
    "params":{
        "username":"USERNAME GOES HERE",
        "password":"PASSWORD GOES HERE",
          "privateKey":"PrivateKey-ewoqjP7PxY4yr3iLTpLisriqt94hdyDFNgchSxGGztUrTXtNN"
    },
    "id": 1
}'
```

就这样！您的Avalanche本地版本已启动并运行。它的默认区块链：[X-链](https://avalanche.gitbook.io/avalanche/learn/platform-overview#exchange-chain-x-chain), [C-链](https://avalanche.gitbook.io/avalanche/learn/platform-overview#contract-chain-c-chain), 和 [P-链](https://avalanche.gitbook.io/avalanche/learn/platform-overview#platform-chain-p-chain)。主网是唯一的子网。

您可以在网络中添加更多节点。但需记住，`db-dir`, `http-port` ，和 `staking-port` 的赋值必须是唯一的。

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU0MTI0ODk4MF19
-->