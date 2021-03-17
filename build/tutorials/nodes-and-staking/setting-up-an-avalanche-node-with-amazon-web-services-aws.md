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

给新的安全组添加一个标签，其键为 `Name` ，值为`Avalanche Security Group`. This will enable us to know what this security group is when we see it in the list of security groups.

![Tag the security group so you can identify it later.](https://miro.medium.com/max/961/1*QehD3uyplkb4RPxddP1qkg.png)

Click `Create security group`. You should see the new security group in the list of security groups.

## Launch an EC2 Instance <a id="0682"></a>

Now you're ready to launch an EC2 instance. Go to the EC2 Dashboard and select **Launch instance**.

![Select &quot;Launch Instance.&quot;](https://miro.medium.com/max/813/1*zsawPDMBFlonC_7kg060wQ.png)

Select **Ubuntu 20.04 LTS \(HVM\), SSD Volume Type** for the operating system.

![Select Ubuntu 20.04 LTS.](https://miro.medium.com/max/1591/1*u438irkY1UoRGHO6v76jRw.png)

Next, choose your instance type. This defines the hardware specifications of the cloud instance. In this tutorial we set up a **c5.large**. This should be more than powerful enough since Avalanche is a lightweight consensus protocol. To create a c5.large instance, select the **Compute-optimized** option from the filter drop-down menu.

![Filter by compute optimized.](https://miro.medium.com/max/595/1*tLVhk8BUXVShgm8XHOzmCQ.png)

Select the checkbox next to the c5.large instance in the table.

![Select c5.large.](https://miro.medium.com/max/883/1*YSmQYAGvwJmKEFg0iA60aQ.png)

Click the **Next: Configure Instance Details** button in the bottom right-hand corner.

![](https://miro.medium.com/max/575/1*LdOFvctYF3HkFxmyNGDGSg.png)

The instance details can stay as their defaults.

### Optional: Using Spot Instances or Reserved Instances <a id="c99a"></a>

By default, you will be charged hourly for running your EC2 instance. There are two ways you may be able to pay less for your EC2.

The first is by launching your EC2 as a **Spot Instance**. Spot instances are instances that are not guaranteed to always be up, but which cost less on average than persistent instances. Spot instances use a supply-and-demand market price structure. As demand for instances goes up, the price for a spot instance goes up. You can set a maximum price you’re willing to pay for the spot instance. You may be able to save a significant amount of money, with the caveat that your EC2 instance may stop if the price increases. Do your own research before selecting this option to determine if the interruption frequency at your maximum price justifies the cost savings. If you choose to use a spot instance, be sure to set the interruption behavior to **Stop**, not **Terminate,** and check the **Persistent Request** option.

The other way you could save money is by using a **Reserved Instance**. With a reserved instance, you pay upfront for an entire year of EC2 usage, and receive a lower per-hour rate in exchange for locking in. If you intend to run a node for a long time and don't want to risk service interruptions, this is a good option to save money. Again, do your own research before selecting this option.

### Add Storage, Tags, Security Group <a id="dbf5"></a>

Click the **Next: Add Storage** button in the bottom right corner of the screen.

You need to add space to your instance's disk. We use 100 GB in this example. The Avalanche database will continually grow until pruning is implemented , so it’s safer to have a larger hard drive allocation for now.

![Select 100 GB for the disk size.](../../../.gitbook/assets/add-storage.png)

Click **Next: Add Tags** in the bottom right corner of the screen to add tags to the instance. Tags enable us to associate metadata with our instance. Add a tag with key `Name` and value `My Avalanche Node`. This will make it clear what this instance is on your list of EC2 instances.

![Add a tag with key &quot;Name&quot; and value &quot;My Avalanche Node.&quot;](https://miro.medium.com/max/1295/1*Ov1MfCZuHRzWl7YATKYDwg.png)

Now assign the security group created earlier to the instance. Choose **Select an existing security group** and choose the security group created earlier.

![Choose the security group created earlier.](../../../.gitbook/assets/configure-security-group.png)

Finally, click **Review and Launch** in the bottom right. A review page will show the details of the instance you're about to launch. Review those, and if all looks good, click the blue **Launch** button in the bottom right corner of the screen.

You'll be asked to select a key pair for this instance. Select **Choose an existing key pair** and then select the `avalanche` key pair you made earlier in the tutorial. Check the box acknowledging that you have access to the `.pem` or `.ppk` file created earlier \(make sure you've backed it up!\) and then click **Launch Instances**.

![Use the key pair created earlier.](https://miro.medium.com/max/700/1*isN2Z7Y39JgoBAaDZ75x-g.png)

You should see a new pop up that confirms the instance is launching!

![Your instance is launching!](https://miro.medium.com/max/727/1*QEmh9Kpn1RbHmoKLHRpTPQ.png)

### Assign an Elastic IP

By default, your instance will not have a fixed IP. Let's give it a fixed IP through AWS's Elastic IP service. Go back to the EC2 dashboard. Under **Network & Security,** select **Elastic IPs**.

![Select &quot;Elastic IPs&quot; under &quot;Network &amp; Security.&quot;](https://miro.medium.com/max/192/1*BGm6pR_LV9QnZxoWJ7TgJw.png)

Select **Allocate Elastic IP address**.

![Select &quot;Allocate Elastic IP address.&quot;](https://miro.medium.com/max/503/1*pjDWA9ybZBKnEr1JTg_Mmw.png)

Select the region your instance is running in, and choose to use Amazon’s pool of IPv4 addresses. Click **Allocate**.

![Settings for the Elastic IP.](https://miro.medium.com/max/840/1*hL5TtBcD_kR71OGYLQnyBg.png)

Select the Elastic IP you just created from the Elastic IP manager. From the **Actions** drop-down, choose **Associate Elastic IP address**.

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
eyJoaXN0b3J5IjpbLTIwNDk3MTM2NzVdfQ==
-->