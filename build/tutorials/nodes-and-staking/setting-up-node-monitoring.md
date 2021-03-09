# 运行Avalanche节点监测

_感谢社区成员Jovica Popović编写了此教程。如有需要，您可以通过我们的_ [_Discord_](https://chat.avax.network) _来联系他_

## 简介

本教程的前提是您在节点上运行了Ubuntu 18.04或20.04\( 本教程的Mac OS X版本即将发布\)。

本教程会展示如何设置基础设施以监测 [AvalancheGo](https://github.com/ava-labs/avalanchego)的实例。

我们将使用：

* [Prometheus](https://prometheus.io/) 收集和储存数据
* [node\_exporter](https://github.com/prometheus/node_exporter) 获取计算机相关信息，
* AvalancheGo’s [指标API](https://docs.avax.network/v1.0/en/api/metrics/) 获取节点相关信息
* [Grafana](https://grafana.com/) 可视化仪表板上的数据。

先决条件：

* 一个运行中的AvalancheGo节点
* 运行节点的计算机的外壳访问
* 计算机的管理员特权

### **警告：安全性**

{% hint style="danger" %}
此处所述系统 **不应**向公共互联网开放。如图所示，Prometheus和Grafana均严禁未经允许的访问。请确保而这均只能通过安全代理、本地网络或VPN访问。本教程不包括其设置，但操作时要谨慎。如果安全操作有问题，则可能使攻击者获得对您节点的控制权！您自行负责遵循适当的安全做法。
{% endhint %}

### 出资

Grafana仪表盘的基础取自 [ColmenaLabs](https://blog.colmenalabs.org/index.html)的优秀人士，目前已不提供。如果您对如何改进本教程有任何想法和建议，请在[Github](https://github.com/ava-labs)上说明、发布问题或发出请求。

## 设置Prometheus

首先，我们需要添加一个系统用户账号并创建目录 \(您会需要超级用户凭据\):

```cpp
sudo useradd -M -r -s /bin/false prometheus
```

```cpp
sudo mkdir /etc/prometheus /var/lib/prometheus
```

下一步，从 [下载页面](https://prometheus.io/download/)获取Prometheus最新版本的链接 \(请确保您选择了合适的处理器架构\)，并使用wget下载并使用tar对存档进行解压缩：

```cpp
mkdir -p /tmp/prometheus && cd /tmp/prometheus
```

```cpp
wget https://github.com/prometheus/prometheus/releases/download/v2.21.0/prometheus-2.21.0.linux-amd64.tar.gz
```

```cpp
tar xvf prometheus-2.21.0.linux-amd64.tar.gz
```

```cpp
cd prometheus-2.21.0.linux-amd64
```

下一步，我们需要移动二进制文件，设置所有权，并将配置文件移动到适当位置：

```cpp
sudo cp {prometheus,promtool} /usr/local/bin/
```

```cpp
sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
```

```cpp
sudo chown -R prometheus:prometheus /etc/prometheus
```

```cpp
sudo chown prometheus:prometheus /var/lib/prometheus
```

```cpp
sudo cp -r {consoles,console_libraries} /etc/prometheus/
```

```cpp
sudo cp prometheus.yml /etc/prometheus/
```

`/etc/prometheus`是用于配置，而`/var/lib/prometheus` 是用于数据。

请对Prometheus进行设置，以将之作为系统服务运行。执行**:**

```cpp
sudo nano /etc/systemd/system/prometheus.service
```

\(或在您选择的文本编辑器中打开该文件\)，并输入以下配置：

```cpp
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus   --config.file=/etc/prometheus/prometheus.yml   --storage.tsdb.path=/var/lib/prometheus   --web.console.templates=/
etc/prometheus/consoles   --web.console.libraries=/etc/prometheus/console_libraries   --web.listen-address=0.0.0.0:9090   --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

保存文件。现在，我们可以将Prometheus作为系统服务运行：

```cpp
sudo systemctl daemon-reload
```

```cpp
sudo systemctl start prometheus
```

```cpp
sudo systemctl enable prometheus
```

Prometheus当前应正在运行中。如需确认，我们可以进行检查：
```cpp
systemctl status prometheus
```

应生成如下结果：

```cpp
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2020-04-01 19:23:53 CEST; 5 months 12 days ago
       Docs: https://prometheus.io/docs/introduction/overview/
   Main PID: 1767 (prometheus)
      Tasks: 12 (limit: 9255)
     CGroup: /system.slice/prometheus.service
             └─1767 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --web.console.templa>

Sep 13 13:00:04 ubuntu prometheus[1767]: level=info ts=2020-09-13T11:00:04.744Z caller=head.go:792 component=tsdb msg="Head GC completed" duration=13.6>
Sep 13 13:00:05 ubuntu prometheus[1767]: level=info ts=2020-09-13T11:00:05.263Z caller=head.go:869 component=tsdb msg="WAL checkpoint complete" first=9>
Sep 13 15:00:04 ubuntu prometheus[1767]: level=info ts=2020-09-13T13:00:04.776Z caller=compact.go:495 component=tsdb msg="write block" mint=15999912000>
...
```

您还可以检查Prometheus网络界面 `http://your-node-host-ip:9090/`

{% hint style="warning" %}
如果防火墙处于**.**，则您可能需要进行`sudo ufw allow 9090/tcp` 
{% endhint %}

## 安装Grafana

使用Ubuntu来设置Grafana项目存储库：

```cpp
sudo apt-get install -y apt-transport-https
```

```cpp
sudo apt-get install -y software-properties-common wget
```

```cpp
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

```cpp
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

如需安装Grafana：

```cpp
sudo apt-get update
```

```cpp
sudo apt-get install grafana
```

将其作为一项服务进行配置：

```cpp
sudo systemctl daemon-reload
```

```cpp
sudo systemctl start grafana-server
```

```cpp
sudo systemctl enable grafana-server.service
```

如需确认其运行正常：

```text
sudo systemctl status grafana-server
```

grafana应显示 `active`。Grafana当前应在 `http://your-node-host-ip:3000/`上可用。

{% hint style="warning" %}
如果防火墙处于**.**，则您可能需要进行`sudo ufw allow 3000/tcp` 
{% endhint %}

使用管理员/管理员用户名/密码登录，并设置一个新的安全密码。目前，我们需要将Grafana连接到我们的数据源Prometheus。

在Grafana的网络界面上：

* 转到左侧菜单上的配置并选择数据源。
* 点击添加数据源。
* 选择Prometheus。
* 在表格中输入姓名 \(Prometheus 会进行\)，将`http://localhost:9090` 作为URL。
* 点击`Save & Test`
* 查看“Data source is working” 的绿色信息。

## 设置节点\_输出点

除了AvalancheGo的指标以外，让我们再设置计算机本身的监测，因此我们可以检查CPU、内存、网络和磁盘使用情况，并了解任何异常情况。为此，我们会使用节点\_输出点，即Prometheus插件。

获取最新版本：
```text
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest | grep browser_download_url | grep linux-amd64 |  cut -d '"' -f 4 | wget -qi -
```

如果您有不同的架构（例如，RaspberryPi是`linux-arm64`），请更改`linux-amd64` 。解压并移动可执行文件。

```cpp
tar xvf node_exporter-1.0.1.linux-amd64.tar.gz
```

```cpp
sudo mv node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin
```

```cpp
node_exporter --version
```

然后我们将节点\_输出点添加为一项服务。请务必：

```cpp
sudo nano /etc/systemd/system/node_exporter.service
```

\(或在您选择的文本编辑器中打开该文件\) 并将其填满：

```cpp
[Unit]
Description=Prometheus
Documentation=https://github.com/prometheus/node_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/node_exporter \
    --collector.cpu \
    --collector.diskstats \
    --collector.filesystem \
    --collector.loadavg \
    --collector.meminfo \
    --collector.filefd \
    --collector.netdev \
    --collector.stat \
    --collector.netstat \
    --collector.systemd \
    --collector.uname \
    --collector.vmstat \
    --collector.time \
    --collector.mdadm \
    --collector.zfs \
    --collector.tcpstat \
    --collector.bonding \
    --collector.hwmon \
    --collector.arp \
    --web.listen-address=:9100 \
    --web.telemetry-path="/metrics"

[Install]
WantedBy=multi-user.target
```

这配置了节点\_输出点，以收集我们可能感兴趣的各种数据。开启服务，并在启动时启用它：

```cpp
sudo systemctl start node_exporter
```

```cpp
sudo systemctl enable node_exporter
```

```cpp
sudo systemctl status node_exporter
```

目前，我们已经准备好了。

## 配置AvalancheGo和节点\_输出点Prometheus工作任务
请确保您的AvalancheGo节点正在与适当的 [command line arguments](../../references/command-line-interface.md)一同运行。 The metrics API must be enabled \(by default, it is\)。指标API必须启用\（默认情况下\)。如果您使用CLI参数 `--http-host` 从主机外部进行API调用，请记下API侦听的地址。

现在我们需要定义一项合适的Prometheus工作任务。让我们编辑Prometheus配置：

请执行：

```cpp
sudo nano /etc/prometheus/prometheus.yml
```

\(或在您选择的文本编辑器中打开该文件\) 并附加至结尾：

```cpp
  - job_name: 'avalanchego'
    metrics_path: '/ext/metrics'
    static_configs:
      - targets: ['<your-host-ip>:9650']

  - job_name: 'avalanchego-machine'
    static_configs:
      - targets: ['<your-host-ip>:9100']
        labels:
          alias: 'machine'
```

**缩进很重要**。请确保 `-job_name` 与现有 `-job_name entry`对齐，其他行也适当缩进。基于您的节点配置方式，请确保您使用了正确的主机IP，或`localhost`, depending on how your node is configured.

Save the config file and restart Prometheus:

```cpp
sudo systemctl restart prometheus
```

Check Prometheus web interface on `http://your-node-host-ip:9090/targets`. You should see three targets enabled:

* Prometheus
* avalanchego
* avalanchego-machine

Open Grafana; you can now create a dashboard using any of those sources. You can also use [the preconfigured dashboards](https://github.com/ava-labs/node-monitoring/tree/master/dashboards).

To import the preconfigured dashboard:

* Open Grafana’s web interface
* Click `+` on the left toolbar
* Select `Import JSON` and then upload the JSON file

That’s it! You may now marvel at all the things your node does. Woohoo!

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg5ODM2MzQ1MSwtNjIzOTExMjk5XX0=
-->