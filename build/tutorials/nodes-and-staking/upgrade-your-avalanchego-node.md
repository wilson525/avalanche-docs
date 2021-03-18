# AvalancheGo节点升级

## **节点备份**

升级节点之前，建议您将用于识别网络上的节点的质押者文件进行备份。在默认安装中，您可以通过运行以下指令进行拷贝：

```text
cd
cp ~/.avalanchego/staking/staker.crt .
cp ~/.avalanchego/staking/staker.key .
```

然后下载 `staker.crt` 和 `staker.key` 文件，把它们保存在安全隐秘的地方。如果节点或运行节点的机器发生任何问题，可以使用这些文件重新创建出所有的节点。

如果您使用这些节点进行开发，并且在节点上有密钥库用户，则这些客户也需要备份。

## 使用安装程序脚本安装的节点

如果您的节点是通过 [安装程序脚本](set-up-node-with-installer.md)安装的，只需再次运行安装程序脚本，即可升级节点。

```text
./avalanchego-installer.sh
```

它会检测到您已经安装了AvalancheGo：

```text
AvalancheGo installer
---------------------
Preparing environment...
Found 64bit Intel/AMD architecture...
Found AvalancheGo systemd service already installed, switching to upgrade mode.
Stopping service...
```

然后它会将您的节点升级到最新版本，完成后，重新启动节点，并打印出最新版本的信息：

```text
Node upgraded, starting service...
New node version:
avalanche/1.1.1 [network=mainnet, database=v1.0.0, commit=f76f1fd5f99736cf468413bbac158d6626f712d2]
Done!
```


这样，您的节点就已升级至最新版本。

如果您的节点是手动安装的，请继续阅读本教程剩余部分。

## **停止运行旧版节点**

备份完成后，您就可以开始升级节点。首先停止运行当前版本。

### 从终端运行的节点

如果您的节点在终端中运行，按 `ctrl+c`停止。

### 作为服务运行的节点

如果您的节点是作为服务运行的，输入以下命令停止：

`sudo systemctl stop avalanchego.service`

\(您的服务可能有不同的命名，如`avalanche.service`或类似的\)

### 后台运行的节点

如果您的节点是在后台运行 \(例如通过运行`nohup`\) 的 ，那么通过运行`ps aux | grep avalanche`来查找运行该节点的进程。将产生如下输出：

```text
ubuntu  6834  0.0  0.0   2828   676 pts/1    S+   19:54   0:00 grep avalanche
ubuntu  2630 26.1  9.4 2459236 753316 ?      Sl   Dec02 1220:52 /home/ubuntu/build/avalanchego
```

在本实例中，第二行显示了节点的信息。注意进程id，在本实例中为`2630`。运行`kill -2 2630`来查找运行该节点的进程。将产生如下输出：


现在我们就可以下载节点的新版本了。您可以下载源代码，然后构建二进制程序，或者您也可以下载预先构建好的二进制程序。二者选其一即可。

下载预先构建好的二进制程序更简单，如果您只是希望运行自己的节点并进行质押，那么建议您下载预先构建好的二进制程序。

如果您是一个希望在Avalanche上进行实验和构建的开发人员，则建议您用 [源代码](upgrade-your-avalanchego-node.md#build-from-source) 构建节点。

## **下载预构建二进制程序**

如果你想下载一个预先构建好的二进制程序，而不是自己构建，请访问我们的[发布页面](https://github.com/ava-labs/avalanchego/releases)，并选择你想要的版本\(可能是最新版本\)

在 `Assets`, select the appropriate file.

For MacOS:  
Download: `avalanchego-macos-<VERSION>.zip`  
Unzip: `unzip avalanchego-macos-<VERSION>.zip`  
The resulting folder, `avalanchego-<VERSION>`, contains the binaries.

For Linux on PCs or cloud providers:  
Download: `avalanchego-linux-amd64-<VERSION>.tar.gz`  
Unzip: `tar -xvf avalanchego-linux-amd64-<VERSION>.tar.gz`  
The resulting folder, `avalanchego-<VERSION>-linux`, contains the binaries.

For Linux on RaspberryPi4 or similar Arm64-based computers:  
Download: `avalanchego-linux-arm64-<VERSION>.tar.gz`  
Unzip: `tar -xvf avalanchego-linux-arm64-<VERSION>.tar.gz`  
The resulting folder, `avalanchego-<VERSION>-linux`, contains the binaries.

You are now ready to run the new version of the node.

### Running the node from terminal

If you are using the pre-built binaries on MacOS:

```cpp
./avalanchego-<VERSION>/build/avalanchego
```

If you are using the pre-built binaries on Linux:

```cpp
./avalanchego-<VERSION>-linux/avalanchego
```

Add `nohup` at the start of the command if you want to run the node in the background.

### Running the node as a service

If you're running the node as a service, you need to replace the old binaries with the new ones.

`cp -r avalanchego-<VERSION>-linux/* <DIRECTORY_WITH_OLD_BINARIES>`

and then restart the service with `sudo systemctl start avalanchego.service`.

## **Build from source**

First clone our Github repo \(you can skip this step if you’ve done this before\):

```text
git clone https://github.com/ava-labs/avalanchego.git
```

Then move to the avalanchego directory:

```text
cd avalanchego
```

Pull the latest code:

```text
git pull
```

Check that your local code is up to date. Do:

```text
git rev-parse HEAD
```

and check that the first 7 characters printed match the Latest commit field on our [Github.](https://github.com/ava-labs/avalanchego)

Now build the binary:

```text
./scripts/build.sh
```

This should print:

```text
Build Successful
```

You can check what version you’re running by doing:

```text
./build/avalanchego --version
```

You can run your node with:

```text
./build/avalanchego
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYzMzU1NzMzMl19
-->