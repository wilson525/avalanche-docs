---
说明：>-
在Avalanche上创建。创建无限制。在Avalanche上创建的开发人员能够轻松地创建功能强大、可靠且安全的应用程序。
---

# 开发人员文档

## 准备开始

{% tabs %}
{% tab title="Coming From Ethereum?" %}
{% page-ref page="build/tutorials/smart-contracts/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask.md" %}

{% page-ref page="build/tutorials/smart-contracts/using-truffle-with-the-avalanche-c-chain.md" %}
{% endtab %}

{% tab title="Avalanche Wallet" %}
{% page-ref page="build/tutorials/nodes-and-staking/staking-avax-by-validating-or-delegating-with-the-avalanche-wallet.md" %}

{% page-ref page="build/tutorials/platform/transfer-avax-between-x-chain-and-p-chain.md" %}

{% page-ref page="build/tutorials/platform/transfer-avax-between-x-chain-and-c-chain.md" %}
{% endtab %}

{% tab title="Staking" %}
{% page-ref page="build/getting-started.md" %}

{% page-ref page="build/tutorials/nodes-and-staking/" %}
{% endtab %}

{% tab title="Advanced" %}
{% page-ref page="build/tutorials/platform/create-a-subnet.md" %}

{% page-ref page="build/tutorials/platform/create-a-new-blockchain.md" %}

{% page-ref page="build/tutorials/smart-digital-assets/create-a-fix-cap-asset.md" %}

{% page-ref page="build/tutorials/smart-digital-assets/creating-a-variable-cap-asset.md" %}

{% page-ref page="build/tutorials/smart-digital-assets/creating-a-nft-part-1.md" %}
{% endtab %}
{% endtabs %}

## Avalanche

[Avalanche](https://avax.network) 是一个用于启动[去中心化应用程序](https://support.avalabs.org/en/articles/4587146-what-is-a-decentralized-application-dapp) 的开放源代码平台和企业[区块链](http://support.avalabs.org/en/articles/4064677-what-is-a-blockchain) 部署于一个互操作、高度可扩展的生态系统中。通过Avalanche，您可以完全控制网络和应用程序层 - 帮助您构建任何您想创建的东西。
Avalanche和其他去中心化网络的一个关键差异在于共识协议。随着时间推移，人们误以为区块链必须是缓慢且不可扩展的。Avalanche协议采用了一种新方法来达成共识，从而实现其强大的安全保证、快速的最终性和高吞吐量，且不影响去中心化。
## Avalanche \(AVAX\) 代币

Avalanche \(AVAX\)代币是Avalanche平台的原始代币，其作用是通过质押、点对点交易和付费来确保网络安全，它还是Avalanche平台上创建的多个子网之间的基本记账单位。`1 nAVAX` 等于 `0.000000001 AVAX`.

## Avalanche 共识协议

![共识对比](.gitbook/assets/consensus-comparison.png)

Avalanche系列中的协议通过反复子抽样投票来运行。当确定是否应接受或拒绝一项[交易](http://support.avalabs.org/en/articles/4587384-what-is-a-transaction)时，[验证程序](http://support.avalabs.org/en/articles/4064704-what-is-a-blockchain-validator)会询问一小个随机子集的验证程序应该接受或拒绝该交易。如果面临询问的验证程序认为该交易无效并表示拒绝，或偏好另一项冲突交易，则该验证程序会回复其认为应拒绝该交易。否则会回复其认为应接受该交易。

如果足够多\(_alpha_ $$α$$\) 的抽样验证程序回复其认为应接受该交易，则验证程序更愿意接受该交易。换言之，当将来该验证程序再次面临有关该交易的询问时，它会回复其认为应接受该交易。与之类似，如果足够多的验证程序回复其认为应拒绝该交易，则验证程序会倾向于拒绝该交易。
验证程序一直重复该抽样过程，直至面临询问的验证程序中的_alpha_ 以相同方式向_beta_连续回复$$β$$轮。

通常情况下，当一项交易不存在冲突时，很快就能做出最终决定。当存在冲突时，诚实的验证程序会迅速集结到冲突交易周围，进入一个正反馈循环，直至所有正确的验证程序都对该交易表示偏好。因此最终会接受非冲突交易，而非拒绝。

![Avalanche共识的工作方式](.gitbook/assets/howavalancheconsensusworks.png)

据保证\(基于系统参数的高可能性\)，如果任一诚实的验证程序接受或拒绝一项交易，则所有诚实的验证程序都会接受或拒绝该交易。

请阅读[whitepaper](https://arxiv.org/pdf/1906.08936.pdf)，了解更多Avalanche共识协议的技术信息。

## Snowman 共识协议

Snowman是一项链优化的共识协议——高吞吐量、高度有序化、十分适配智能合约。Snowman由[Avalanche consensus protocol](./#avalanche-consensus-protocol). Both [P-Chain](learn/platform-overview/#platform-chain-p-chain) and [C-Chain](learn/platform-overview/#contract-chain-c-chain) implement the Snowman consensus protocol.

## Key Features

### Speed

Uses a novel consensus protocol, developed by a team of Cornell computer scientists, and is able to permanently confirm transactions in under 1 second.

### Scalability

Capable of 4,500 transactions per second–an order of magnitude greater than existing blockchains.

### Security

Ensures stronger security guarantees well-above the 51% standard of other networks.

### Flexibility

Easily create custom blockchains and decentralized apps that contain almost any arbitrary logic.

### Sustainability

Uses energy-efficient proof-of-stake consensus algorithm rather than proof-of-work.

### Smart Contract Support

Supports the creation of Solidity smart contracts and your favorite Ethereum tools like Remix, Metamask, Truffle, and more.

### Private and Public Blockchains

Create your own public or private blockchains.

### Designed for Finance

Native support for easily creating and trading digital smart assets with complex, custom rulesets.

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTI0NjE5NTEyXX0=
-->