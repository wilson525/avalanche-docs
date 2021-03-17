# 使用亚马逊网络服务\(AWS\)运行Avalanche节点

## 简介

通过本教程，您会了解如何在 [亚马逊网络服务 \(AWS\)](https://aws.amazon.com/)上设置一个Avalanche节点。AWS等云服务能够很好地确保您的节点高度安全、可用且可访问。

如需启动，您需要：

* AWS账户
* 用于SSH到AWS计算机的终端
* 安全存储和备份文件的地方

本教程假设您的本地计算机拥有Unix系统终端。如果您用的是Windows系统，则您必须调整此处使用的一些命令。

## 登录AWS <a id="ff31"></a>

本文不讨论如何注册AWS，但是亚马逊[here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account)有相关说明

强烈建议您在AWS根用户帐户上设置多重身份验证进行保护。亚马逊 [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-root)对此有相关文件

一旦设置了您的账户，您应创建一个新的EC2实例。EC2是AWS云端的一个虚拟机实例。访问 [AWS 管理控制台](https://console.aws.amazon.com/) 并进入 EC2 仪表盘

![AWS Management Console.png](../../../.gitbook/assets/AWS-Management-Console.png)

如需登录EC2实例，您需要本地计算机上的一个密钥，以确保对实例的访问。首先，创建该密钥，以用于后续EC2实例的分配。在左栏， **网络&安全**项下选择 **关键对**

![Select &quot;Key Pairs&quot; under the &quot;Network &amp; Security&quot; drop-down.](../../../.gitbook/assets/Network-and-Security.png)

选择 **Create key pair** ，以启动密钥对创建向导。
![Select &quot;Create key pair.&quot;](https://miro.medium.com/max/847/1*UZ4L0DGUogCfBq-TZ5U3Kw.png)

将您的密钥命名为 `avalanche`。如果您的本地计算机是MacOS或Linux操作系统，请选择 `pem` 文件格式。如果是Windows系统，请使用 `ppk` 文件格式。或者，您可以为密钥对添加标记，以协助追踪。

![Create a key pair that will later be assigned to your EC2 instance.](https://miro.medium.com/max/827/1*Bo30BXjwPTGpgFtoU9VDBA.png)

点击 `Create key pair`。您应看到一条成功消息，并且密钥文件应下载到您的本地计算机上。如无此文件，您将无法访问EC2实例。**复制此文件并将其放在单独的存储介质（如外部硬盘）上。此文件保密，不得与他人共享**

![Success message after creating a key pair.](https://miro.medium.com/max/534/1*RGpHRWWFjNKMZb7cQTyeWQ.png)

## 创建安全组 <a id="f8df"></a>

AWS安全组定义了什么样的互联网流量可以进入并退出您的EC2实例。将其想象成防火墙。在**Security Groups** 下拉列表中选择 **Network & Security** ，创建一个新的安全组。

![Select &quot;Security Groups&quot; underneath &quot;Network &amp; Security.&quot;](https://miro.medium.com/max/214/1*pFOMpS0HhzcAYbl_VfyWlA.png)

打开安全组面板。点击安全组面板右上角的 **Create security group** 

![Select &quot;Create security group.&quot;](https://miro.medium.com/max/772/1*B0JSYoMBplAtCz2Yb2e1sA.png)

您需要指定哪些入站流量是允许进入的。允许来自您的IP地址的SSH流量，因此您可以登陆您的EC2实例。\(每次您的ISP更改您的IP地址时，您需要修改此规则。如果您的ISP定期更改，则您可以允许来自任何地方的SSH流量进入，以避免频繁修改此规则。\)允许端口9651上的TCP通信，则您的节点可以与网络上的其他节点通信。允许来自IP的端口9650上的TCP通信，则可以对节点进行API调用。 **重要的是，您只允许从您的IP上访问这个端口。**如果允许来自任何地方的入站流量，则这可用作服务攻击向量的拒绝。最后，允许所有出站流量。

![Your inbound and outbound rules should look like this.](../../../.gitbook/assets/inbound-rules.png)

给新的安全组添加一个标签，其键为 `Name` ，值为`Avalanche Security Group`。借此，当我们在安全组列表中看到该安全组时，我们能够知晓它是什么。

![Tag the security group so you can identify it later.](https://miro.medium.com/max/961/1*QehD3uyplkb4RPxddP1qkg.png)

点击 `Create security group`。您应在安全组列表中看到新的安全组。

## 启动EC2实例 <a id="0682"></a>

现在，您可以启动EC2实例。访问EC2仪表盘并选择 **Launch instance**.

![Select &quot;Launch Instance.&quot;](https://miro.medium.com/max/813/1*zsawPDMBFlonC_7kg060wQ.png)

为操作系统选择 **Ubuntu 20.04 LTS \(HVM\), SSD Volume Type** 

![Select Ubuntu 20.04 LTS.](https://miro.medium.com/max/1591/1*u438irkY1UoRGHO6v76jRw.png)

下一步，选择您的实例类型。这定义了云实例的硬件规范。在本教程中，我们设置了一个 **c5.large**. 。由于Avalanche是一个轻量级共识协议，因此这应当已经足够了。如需创建c5.large实例，请从筛选程序下拉菜单中选择 **Compute-optimized**选项。

![Filter by compute optimized.](https://miro.medium.com/max/595/1*tLVhk8BUXVShgm8XHOzmCQ.png)

选择表格中c5.large实例旁的复选框。

![Select c5.large.](https://miro.medium.com/max/883/1*YSmQYAGvwJmKEFg0iA60aQ.png)

击右下角的 **Next: Configure Instance Details** 按钮。

![](https://miro.medium.com/max/575/1*LdOFvctYF3HkFxmyNGDGSg.png)

实例详情可保留为默认值。

### 可选：使用点实例或保留实例 <a id="c99a"></a>

默认情况下，运行EC2实例将按小时计费。如需减少运行EC2的费用，有两种方法。

第一种方法：将EC2作为 **Spot Instance**启动。点实例是并不保证总是能启动的实例，但其平均成本低于持久实例。点实例采用供需市场价格结构。随着实例需求的上升，点实例的价格也上涨。您可以设定一个您愿意为点实例的支付的最高价格。您可以节省大量资金，但需要注意的是，如果价格上涨，您的EC2实例可能会停止。在选择此选项之前，请先自行调研，以确定以您的最高价格计算的中断频率是否能节省成本。如果选择使用点实例，请确保将中断行为设置为 **Stop**而不是 **Terminate,** ，并查看 **Persistent Request** 选项。

第二种省钱方法：使用**Reserved Instance**。您需要为保留实例预付一整年的EC2使用费，并以较低的每小时费率换取锁定。如果您计划长时间运行一个节点，并且不想冒服务中断的风险，那么这是一个很好的省钱选择。同样，选择此选项之前，请自行调研。

### 添加存储、标签、安全组 <a id="dbf5"></a>

点击屏幕右下角的 **Next: Add Storage**

您需要为您的实例磁盘添加空间。本例中，我们使用的是100GB。Avalanche数据库将不断扩大，直至实现精简，所以目前而言更安全的做法是拥有更大的硬盘驱动分配。

![Select 100 GB for the disk size.](../../../.gitbook/assets/add-storage.png)

点击屏幕右下角的 **Next: Add Tags** ，为实例添加标签。通过标签，我们可以将元数据与实例联系起来。添加标签，其键为 `Name` ，值为 `My Avalanche Node`。借此，您能够明确该实例在您的EC2实例列表中是什么。

![Add a tag with key &quot;Name&quot; and value &quot;My Avalanche Node.&quot;](https://miro.medium.com/max/1295/1*Ov1MfCZuHRzWl7YATKYDwg.png)

现在，将之前创建的安全组分配给实例。选择 **Select an existing security group** ，并选择之前创建的安全组。

![Choose the security group created earlier.](../../../.gitbook/assets/configure-security-group.png)

最后，点击右下角的 **Review and Launch** 。查看页面上会显示您将要启动的实例的详情。查看后，如果一切运行良好，则点击屏幕右下角的蓝色 **Launch** 按钮。

我们会要求您为此实例选择一个关键对。选择 **Choose an existing key pair** ，然后选择您之前在教程中创建的`avalanche` 关键对。检查确认您有权访问先前创建的 `.pem` 或 `.ppk` 文件的复选框（请确保您已备份！）然后单击**Launch Instances**

![Use the key pair created earlier.](https://miro.medium.com/max/700/1*isN2Z7Y39JgoBAaDZ75x-g.png)

您应该会看到一个新的弹出窗口，以确认实例正在启动！
![Your instance is launching!](https://miro.medium.com/max/727/1*QEmh9Kpn1RbHmoKLHRpTPQ.png)

### 分配一个弹性IP

默认情况下，您的实例并不拥有固定IP。让我们通过AWS的弹性IP服务给您的实例一个固定IP。返回EC2仪表盘。 **Network & Security,** 项下，选择 **Elastic IPs**.

![Select &quot;Elastic IPs&quot; under &quot;Network &amp; Security.&quot;](https://miro.medium.com/max/192/1*BGm6pR_LV9QnZxoWJ7TgJw.png)

选择 **Allocate Elastic IP address**.

![Select &quot;Allocate Elastic IP address.&quot;](https://miro.medium.com/max/503/1*pjDWA9ybZBKnEr1JTg_Mmw.png)

选择实例运行所在的区域，并选择使用亚马逊的IPv4地址库。点击**Allocate**.

![Settings for the Elastic IP.](https://miro.medium.com/max/840/1*hL5TtBcD_kR71OGYLQnyBg.png)

选择您刚从弹性IP管理器中创建的弹性IP。在 **Actions** 下拉列表中，选择 **Associate Elastic IP address**。

![Under &quot;Actions&quot;, select &quot;Associate Elastic IP address.&quot;](https://miro.medium.com/max/490/1*Mj6N7CllYVJDl_-zcCl-gw.png)

Select the instance you just created. This will associate the new Elastic IP with the instance and give it a public IP address that won't change.

![Assign the Elastic IP to your EC2 instance.](https://miro.medium.com/max/834/1*NW-S4LzL3EC1q2_4AkIPUg.png)

## Set Up AvalancheGo <a id="829e"></a>

Go back to the EC2 Dashboard and select `Running Instances`.

![Go to your running instances.](https://miro.medium.com/max/672/1*CHJZQ7piTCl_nsuEAeWpDw.png)

Select the newly created EC2 instance. This opens a details panel with information about the instance.

![Details about your new instance.](https://miro.medium.com/max/1125/1*3DNT5ecS-Dbf33I_gxKMlg.png)

Copy the `IPv4 Public IP` field to use later. From now on we call this value `PUBLICIP`.

**Remember: the terminal commands below assume you're running Linux. Commands may differ for MacOS or other operating systems. When copy-pasting a command from a code block, copy and paste the entirety of the text in the block.**

Log into the AWS instance from your local machine. Open a terminal \(try shortcut `CTRL + ALT + T`\) and navigate to the directory containing the `.pem` file you downloaded earlier.

Move the `.pem` file to `$HOME/.ssh` \(where `.pem` files generally live\) with:

```bash
mv avalanche.pem ~/.ssh
```

Add it to the SSH agent so that we can use it to SSH into your EC2 instance, and mark it as read-only.

```bash
ssh-add ~/.ssh/avalanche.pem; chmod 400 ~/.ssh/avalanche.pem
```

SSH into the instance. \(Remember to replace `PUBLICIP` with the public IP field from earlier.\)

```text
ssh ubuntu@PUBLICIP
```

If the permissions are **not** set correctly, you will see the following error.

![Make sure you set the permissions correctly.](https://miro.medium.com/max/1065/1*Lfp8o3DTsGfoy2HOOLw3pg.png)

You are now logged into the EC2 instance.

![You&apos;re on the EC2 instance.](https://miro.medium.com/max/1030/1*XNdOvUznKbuuMF5pMf186w.png)

If you have not already done so, update the instance to make sure it has the latest operating system and security updates:

```text
sudo apt update; sudo apt upgrade -y; sudo reboot
```

This also reboots the instance. Wait 5 minutes, then log in again by running this command on your local machine:

```bash
ssh ubuntu@PUBLICIP
```

You're logged into the EC2 instance again. Now we’ll need to set up our Avalanche node. To do this, follow the [Set Up Avalanche Node With Installer](set-up-node-with-installer.md) tutorial which automates the installation process. You will need the `PUBLICIP` we set up earlier.

Your AvalancheGo node should now be running and in the process of bootstrapping, which can take a few hours. To check if it's done, you can issue an API call using `curl`. If you're making the request from the EC2 instance, the request is:

```text
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.isBootstrapped",
    "params": {
        "chain":"X"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

Once the node is finished bootstrapping, the response will be:

```text
{
    "jsonrpc": "2.0",
    "result": {
        "isBootstrapped": true
    },
    "id": 1
}
```

You can continue on, even if AvalancheGo isn't done bootstrapping.

In order to make your node a validator, you'll need its node ID. To get it, run:

```text
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response contains the node ID.

```text
{"jsonrpc":"2.0","result":{"nodeID":"NodeID-DznHmm3o7RkmpLkWMn9NqafH66mqunXbM"},"id":1}
```

In the above example the node ID is`NodeID-DznHmm3o7RkmpLkWMn9NqafH66mqunXbM`. Copy your node ID for later. Your node ID is not a secret, so you can just paste it into a text editor.

AvalancheGo has other APIs, such as the [Health API](../../avalanchego-apis/health-api.md), that may be used to interact with the node. Some APIs are disabled by default. To enable such APIs, modify the ExecStart section of `/etc/systemd/system/avalanchego.service` \(created during the installation process\) to include flags that enable these endpoints. Don't manually enable any APIs unless you have a reason to.

![Some APIs are disabled by default.](https://miro.medium.com/max/881/1*Vm-Uh2yV0pDCVn8zqFw64A.png)

Back up the node's staking key and certificate in case the EC2 instance is corrupted or otherwise unavailable. The node's ID is derived from its staking key and certificate. If you lose your staking key or certificate then your node will get a new node ID, which could cause you to become ineligible for a staking reward if your node is a validator. **It is very strongly advised that you copy your node's staking key and certificate**. The first time you run a node, it will generate a new staking key/certificate pair and store them in directory `/home/ubuntu/.avalanchego/staking`.

Exit out of the SSH instance by running:

```bash
exit
```

Now you're no longer connected to the EC2 instance; you're back on your local machine.

To copy the staking key and certificate to your machine, run the following command. As always, replace `PUBLICIP`.

```text
scp -r ubuntu@PUBLICIP:/home/ubuntu/.avalanchego/staking ~/aws_avalanche_backup
```

Now your staking key and certificate are in directory `~/aws_avalanche_backup` . **The contents of this directory are secret.** You should hold this directory on storage not connected to the internet \(like an external hard drive.\)

### Upgrading Your Node <a id="9ac7"></a>

AvalancheGo is an ongoing project and there are regular version upgrades. Most upgrades are recommended but not required. Advance notice will be given for upgrades that are not backwards compatible. To update your node to the latest version, SSH into your AWS instance as before and run the installer script again.

```text
./avalanchego-installer.sh
```

Your machine is now running the newest AvalancheGo version. To see the status of the AvalancheGo service, run `sudo systemctl status avalanchego.`

## Wrap Up

That's it! You now have an AvalancheGo node running on an AWS EC2 instance. We recommend setting up [node monitoring ](setting-up-node-monitoring.md)for your AvalancheGo node. We also recommend setting up AWS billing alerts so you're not surprised when the bill arrives. If you have feedback on this tutorial, or anything else, send us a message on [Discord](https://chat.avalabs.org).

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNjE2NzA4Ml19
-->