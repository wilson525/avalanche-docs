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

如需添加验证程序，我们需要保持一个节点处于运行中。我们可以使用发布的 [二进制文件](https://github.com/ava-labs/avalanchego/releases/) 设置一个或从 [AvalancheGo源代码](https://github.com/ava-labs/avalanchego)中构建。

使用二进制文件非常简单方便，只要4个步骤您就可以设置好一个验证程序：

* 在[here](https://github.com/ava-labs/avalanchego/releases)下载最新版本tar.gz\( osx和windows的压缩包\)
* 解压成我们选择的文件夹：

\* Linux系统：解压 tar -xvf avalanchego-linux-&lt;VERSION&gt;.tar.gz

\* OSX系统：解压 avalanchego-macos-&lt;VERSION&gt;.zip

\* Windows系统： avalanchego-win-&lt;VERSION&gt;.zip

* 导航到二进制文件目录 cd avalanchego-&lt;VERSION&gt;
* 在Linux和OSX系统中运行二进制文件与 /avalanchego ，在Windows系统中运行AvalancheGo。

我们会让我们的节点与网络的其他部分一同引导并同步，并且我们随时准备开始。

我们会需要节点ID。通过[info API](../../avalanchego-apis/info-api.md)，我们可以找到它

如果您在设置您的节点时需要任何帮助，请通过[Discord](https://chat.avax.network/)加入我们

![Image for post](https://miro.medium.com/max/1600/0*6hZSaT651Dd7R4bL)

填完字段并确认！

![Image for post](https://miro.medium.com/max/1600/0*cy61ZMDY5veMvCZj)

仔细检查细节，并再次点击“确认”！

![Image for post](https://miro.medium.com/max/1600/0*f3GlN03He6TFkOV7)

恭喜。您现在正在验证Avalanche主网！

### **第3步B：添加委托程序！** <a id="59bd"></a>

![Image for post](https://miro.medium.com/max/1600/0*f-wXi2SiSm4eBmHt)

从可用网络验证程序列表中选择您想要委托代币的验证程序。

![Image for post](https://miro.medium.com/max/1600/0*uNnT2PtjCslRKFbF)

指明您的质押期限和质押金额。注意所选验证程序的结束时间。您的委托期不得晚于验证程序设置的结束日期。

![Image for post](https://miro.medium.com/max/1600/0*M_6_7L9jtYuPTp-A)

确认详情！

![Image for post](https://miro.medium.com/max/1600/0*Silj8-uZTm5g9xSi)

恭喜。现在您可以委派Avalanche主网了！

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMjg0MjU4MzNdfQ==
-->