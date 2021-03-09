# 使用Avalanche钱包，通过验证或委托来质押AVAX

## **简介** <a id="001f"></a>

Avalanche钱包是一个网络应用程序，无中介软件或任何类型的服务器通信。Avalanche钱包是采用Vue JS编写的，既可以线上访问，也可以本地编译和运行。

Avalanche钱包可以从 [这里](https://wallet.avax.network/)访问。
Avalanche钱包源代码可以从[这里](https://github.com/ava-labs/avalanche-wallet)找到。

**来质押吧！**

### **第1步——打开Avalanche钱包** <a id="552d"></a>

![Image for post](https://miro.medium.com/max/1552/0*tpBIOjLdppuNKMjA)

您可以使用您的关键字、密钥库文件或Ledger Nano S\(即将推出!\)来访问您的钱包。
### **第2步——导航到“赚钱”章节** <a id="dc5a"></a>

![Image for post](https://miro.medium.com/max/1504/0*XTh3nZzBI1bkLbwO)

**如需质押，您需要在** [**平台链 \(P-Chain\)**](../../../learn/platform-overview/#platform-chain-p-chain)**上提供资金！如果您的资金处于** [**交换链 \(X-Chain\)**](../../../learn/platform-overview/#exchange-chain-x-chain)**，我们将需要通过启动跨链转账将其转到P链上。如果您的代币已锁定，则您的代币已处于P链上，所以您不需要将资金从X链跨链转账到P链上。**

![Image for post](https://miro.medium.com/max/1522/0*xKAf0nXSzqIdmBDg)

输入您希望转账到P链上的金额，并通过点击下方“转账”按钮来完成转账。

![Image for post](https://miro.medium.com/max/1488/0*aremeYNYtKP5nGPx)

就是这！

![Image for post](https://miro.medium.com/max/1512/0*XP8f8CISy-LJ_Lc3)

目前，我们已经将资金准备好质押在P链上。下一步，您可以给您的钱包添加一个验证程序或委托程序。

### **第3步A：成为验证程序！** <a id="60f0"></a>

如需添加验证程序，我们需要保持一个节点处于运行中。我们可以使用发布的 [二进制文件](https://github.com/ava-labs/avalanchego/releases/) or build them from the [AvalancheGo source code](https://github.com/ava-labs/avalanchego).

Using the binaries is easy and convenient and sets you up to be a validator in 4 steps:

* Download the latest release tar.gz \(zip for osx and windows\) found [here](https://github.com/ava-labs/avalanchego/releases)
* Unpack into a folder of our choosing:

\* Linux: tar -xvf avalanchego-linux-&lt;VERSION&gt;.tar.gz

\* OSX: unzip avalanchego-macos-&lt;VERSION&gt;.zip

\* Windows: unzip avalanchego-win-&lt;VERSION&gt;.zip

* Navigate to the binaries directory cd avalanchego-&lt;VERSION&gt;
* Run the binary with ./avalanchego on Linux and OSX and AvalancheGo on Windows

We will let our node bootstrap and sync with the rest with the network, and we are ready to roll.

We will need our Node ID. Let’s find that using the [info API](../../avalanchego-apis/info-api.md)!

If you need any help setting up your node, join us on [Discord](https://chat.avax.network/).

![Image for post](https://miro.medium.com/max/1600/0*6hZSaT651Dd7R4bL)

Fill the fields and confirm!

![Image for post](https://miro.medium.com/max/1600/0*cy61ZMDY5veMvCZj)

Carefully check the details, and click “Confirm” again!

![Image for post](https://miro.medium.com/max/1600/0*f3GlN03He6TFkOV7)

Congratulations. You are now validating the Avalanche Primary Network!

### **Step 3B: Add a Delegator!** <a id="59bd"></a>

![Image for post](https://miro.medium.com/max/1600/0*f-wXi2SiSm4eBmHt)

Select a validator you want to delegate your tokens with from the list of active network validators.

![Image for post](https://miro.medium.com/max/1600/0*uNnT2PtjCslRKFbF)

Specify your staking period and stake amount. Pay attention to the end time of the selected validator. Your delegation period cannot be set to end past the end date that the validator has set.

![Image for post](https://miro.medium.com/max/1600/0*M_6_7L9jtYuPTp-A)

Confirm the details!

![Image for post](https://miro.medium.com/max/1600/0*Silj8-uZTm5g9xSi)

Congratulations. You are now delegating the Avalanche Primary Network!

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkzOTY2MDU4N119
-->