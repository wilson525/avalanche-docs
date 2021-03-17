# 通过安装脚本在Linux系统上运行Avalanche节点

我们有一个shell \(bash\) 脚本，用于在您的计算机上安装AvalancheGo。此脚本几分钟内即可设置完整的、正在运行的节点，且只需少量用户输入。

## 在您开始之前

此安装脚本假定：

* 操作系统：Ubuntu 18.04或20.04 \(很抱歉，尚不支持MacOS和Windows系统\)
* AvalancheGo未运行且尚未作为一项服务安装
* 运行脚本的用户拥有超级用户权限\(可以运行“sudo”\)
* 
### 环境考虑
如果您运行不同类型的Linux，脚本可能无法按预期工作。它假定 `systemd` 用于运行系统服务。其他类型的Linux可能会使用其他脚本，或者可能在脚本假定的不同位置有文件。

如果您已经在电脑上运行一个节点，那么请在运行此脚本前停止运行。

#### 终端上的节点运行

如果您的节点在终端上运行，请按下 `ctrl+C`，停止运行。

#### 作为一项服务来运行节点

如果您的节点已经作为一项服务在运行，那么您可能不需要此脚本。您已经可以开始了。
#### 后台运行的节点
如果您的节点正在后台运行 \(例如，通过与`nohup`, 来运行\) ，那么就通过运行 `ps aux | grep avalanche`来查找运行节点的进程。其可能输出有：

```text
ubuntu  6834  0.0  0.0   2828   676 pts/1    S+   19:54   0:00 grep avalanche
ubuntu  2630 26.1  9.4 2459236 753316 ?      Sl   Dec02 1220:52 /home/ubuntu/build/avalanchego
```

查找不包含`grep` 本例中，不包含“grep”的是第二行。其中展示了您的节点信息。请注意本例中的进程ID是`2630`。通过运行 `kill -2 2630`.

####节点工作文档

如果您曾在此电脑上运行过AvalancheGo节点，则您会在`$HOME/.avalanchego` 目录下保存本地节点文档。此类文档不会受到干扰，此脚本设置的节点将以之前的同一身份和状态继续运行。换言之，为了您的节点安全，请备份 `$HOME/.avalanchego/staking` 目录下的`staker.crt` 和 `staker.key`文件，并将其保存到其他安全地方。如有需要，您可以使用上述文档在另一台电脑上重新创建您的节点。

### 联网注意事项
T如需成功运行，AvalancheGo需要通过网络接口`9651`进行联网。在您进行安装之前，您需要确定您的节点将要运行的联网环境。

#### 在云供应商上运行

如果您的节点正在云供应商计算机实例上运行，则该节点会有一个固定IP。找出该固定IP，或如果您并未准备好，请设置。此脚本会尝试自行找出IP，但是这并非所有环境下都可行，因此您需要检查IP或自行输入。

#### 在家庭连接上运行

I如果您正在一台住宅网络连接的计算机上运行节点，则您拥有一个动态IP；换言之，您的IP会定期变化。安装脚本会针对此情况合理配置节点。但是，如果是家庭连接，您需要设置从互联网到节点所安装计算机转发 `9651` 端口的入站端口。

由于型号和路由器配置过多，因此我们无法就具体应该做什么给出意见，但是可以在\(如 [这里](https://www.noip.com/support/knowledgebase/general-port-forwarding-guide/)，或 [这里](https://www.howtogeek.com/66214/how-to-forward-ports-on-your-router/) \)找到线上指南，您的服务供应商支持可能也有所帮助。

## 运行脚本


所以，既然您已经准备好了系统和信息，那就开始吧。

如需下载并运行脚本，请在终端输入下列信息：

```text
wget https://raw.githubusercontent.com/ava-labs/avalanche-docs/master/scripts/avalanchego-installer.sh;\
chmod 755 avalanchego-installer.sh;\
./avalanchego-installer.sh
```

让我们开始吧！输出应该看起来是这样的：

```text
AvalancheGo installer
---------------------
Preparing environment...
Found arm64 architecture...
Looking for the latest arm64 build...
Will attempt to download:
 https://github.com/ava-labs/avalanchego/releases/download/v1.1.1/avalanchego-linux-arm64-v1.1.1.tar.gz
avalanchego-linux-arm64-v1.1.1.tar.gz 100%[=========================================================================>]  29.83M  75.8MB/s    in 0.4s
2020-12-28 14:57:47 URL:https://github-production-release-asset-2e65be.s3.amazonaws.com/246387644/f4d27b00-4161-11eb-8fb2-156a992fd2c8?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20201228%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20201228T145747Z&X-Amz-Expires=300&X-Amz-Signature=ea838877f39ae940a37a076137c4c2689494c7e683cb95a5a4714c062e6ba018&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=246387644&response-content-disposition=attachment%3B%20filename%3Davalanchego-linux-arm64-v1.1.1.tar.gz&response-content-type=application%2Foctet-stream [31283052/31283052] -> "avalanchego-linux-arm64-v1.1.1.tar.gz" [1]
Unpacking node files...
avalanchego-v1.1.1/plugins/
avalanchego-v1.1.1/plugins/evm
avalanchego-v1.1.1/avalanchego
Node files unpacked into /home/ubuntu/avalanche-node
```

然后，脚本会提示您输入有关网络环境的信息：

```text
To complete the setup some networking information is needed.
Where is the node installed:
1) residential network (dynamic IP)
2) cloud provider (static IP)
Enter your connection type [1,2]:
```

如果您是动态IP，请输入`1` ，如果您是固定IP，请输入 `2` 。如果您是固定IP，则它会尝试自动检测IP并询问确认。
```text
Detected '3.15.152.14' as your public IP. Is this correct? [y,n]:
```

按 `y` 确认，如果检测到的IP有误\(或为空\)，然后在下一个弹窗中输入正确IP。

然后，脚本会继续创建系统服务，并在启动服务时结束。

```text
Installing service with public IP: 3.15.152.14
Created symlink /etc/systemd/system/multi-user.target.wants/avalanchego.service → /etc/systemd/system/avalanchego.service.

Done!

Your node should now be bootstrapping on the main net.
To check that the service is running use the following command (q to exit):
sudo systemctl status avalanchego
To follow the log use (ctrl+C to stop):
sudo journalctl -u avalanchego -f

Reach us over on https://chat.avax.network if you're having problems.
```

脚本完成，您应该会再次看到系统弹窗。

## 安装后

AvalancheGo应在后台作为一项服务运行。您可以查看它是否正在运行：

```text
sudo systemctl status avalanchego
```

它会打印节点的最新日志，如下所示：

```text
● avalanchego.service - AvalancheGo systemd service
Loaded: loaded (/etc/systemd/system/avalanchego.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2021-01-05 10:38:21 UTC; 51s ago
Main PID: 2142 (avalanchego)
Tasks: 8 (limit: 4495)
Memory: 223.0M
CGroup: /system.slice/avalanchego.service
└─2142 /home/ubuntu/avalanche-node/avalanchego --plugin-dir=/home/ubuntu/avalanche-node/plugins --dynamic-public-ip=opendns --http-host=

Jan 05 10:38:45 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:45] <P Chain> avalanchego/vms/platformvm/vm.go#322: initializing last accepted block as 2FUFPVPxbTpKNn39moGSzsmGroYES4NZRdw3mJgNvMkMiMHJ9e
Jan 05 10:38:45 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:45] <P Chain> avalanchego/snow/engine/snowman/transitive.go#58: initializing consensus engine
Jan 05 10:38:45 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:45] avalanchego/api/server.go#143: adding route /ext/bc/11111111111111111111111111111111LpoYY
Jan 05 10:38:45 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:45] avalanchego/api/server.go#88: HTTP API server listening on ":9650"
Jan 05 10:38:58 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:58] <P Chain> avalanchego/snow/engine/common/bootstrapper.go#185: Bootstrapping started syncing with 1 vertices in the accepted frontier
Jan 05 10:39:02 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:39:02] <P Chain> avalanchego/snow/engine/snowman/bootstrap/bootstrapper.go#210: fetched 2500 blocks
Jan 05 10:39:04 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:39:04] <P Chain> avalanchego/snow/engine/snowman/bootstrap/bootstrapper.go#210: fetched 5000 blocks
Jan 05 10:39:06 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:39:06] <P Chain> avalanchego/snow/engine/snowman/bootstrap/bootstrapper.go#210: fetched 7500 blocks
Jan 05 10:39:09 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:39:09] <P Chain> avalanchego/snow/engine/snowman/bootstrap/bootstrapper.go#210: fetched 10000 blocks
Jan 05 10:39:11 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:39:11] <P Chain> avalanchego/snow/engine/snowman/bootstrap/bootstrapper.go#210: fetched 12500 blocks
```

请注意，`active (running)` 说明服务运行正常。您可能需要按 `q` 来返回命令弹窗。

如需找出您的节点ID，以在网络中辨识您的节点，请运行如下命令：

```text
sudo journalctl -u avalanchego | grep "node's ID"
```

其输出应是：

```text
Jan 05 10:38:38 ip-172-31-30-64 avalanchego[2142]: INFO [01-05|10:38:38] avalanchego/node/node.go#428: Set node's ID to 6seStrauyCnVV7NEVwRbfaT9B6EnXEzfY
```

为 `NodeID-` 预设要得到的值，例如`NodeID-6seStrauyCnVV7NEVwRbfaT9B6EnXEzfY`保存；质押或查找您的ID时这是必需的。

您的节点现在应该正在引导过程中。您可以通过发出以下命令来监视进程：

```text
sudo journalctl -u avalanchego -f
```

当您想要停止读取节点输出时，请按下 `ctrl+C` 

## 停止节点

如需停止AvalancheGo，请运行：
```text
sudo systemctl stop avalanchego
```

如需再次启动，请运行：

```text
sudo systemctl start avalanchego
```

## 节点升级

AvalancheGo is an ongoing project and there are regular version upgrades. Most upgrades are recommended but not required. Advance notice will be given for upgrades that are not backwards compatible. When a new version of the node is released, you will notice log lines like:

```text
Jan 08 10:26:45 ip-172-31-16-229 avalanchego[6335]: INFO [01-08|10:26:45] avalanchego/network/peer.go#526: beacon 9CkG9MBNavnw7EVSRsuFr7ws9gascDQy3 attempting to connect with newer version avalanche/1.1.1. You may want to update your client
```

It is recommended to always upgrade to the latest version, because new versions bring bug fixes, new features and upgrades.

To upgrade your node, just run the installer script again:

```text
./avalanchego-installer.sh
```

It will detect that you already have AvalancheGo installed:

```text
AvalancheGo installer
---------------------
Preparing environment...
Found 64bit Intel/AMD architecture...
Found AvalancheGo systemd service already installed, switching to upgrade mode.
Stopping service...
```

It will then upgrade your node to the latest version, and after it's done, start the node back up, and print out the information about the latest version:

```text
Node upgraded, starting service...
New node version:
avalanche/1.1.1 [network=mainnet, database=v1.0.0, commit=f76f1fd5f99736cf468413bbac158d6626f712d2]
Done!
```

## What next?

That's it, you're running an AvalancheGo node! Congratulations! Let us know you did it on our [Twitter](https://twitter.com/avalancheavax), [Telegram](https://t.me/avalancheavax) or [Reddit](https://t.me/avalancheavax)!

If you're on a residential network \(dynamic IP\), don't forget to set up port forwarding. If you're on a cloud service provider, you're good to go.

Now you can [interact with your node](../../avalanchego-apis/issuing-api-calls.md), [stake your tokens](staking-avax-by-validating-or-delegating-with-the-avalanche-wallet.md), or level up your installation by setting up [node monitoring](setting-up-node-monitoring.md) to get a better insight into what your node is doing.

If you have any questions, or need help, feel free to contact us on our [Discord](https://chat.avalabs.org/) server.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA4NzUzNTEzNV19
-->