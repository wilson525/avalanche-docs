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

下一步，从 [downloads page](https://prometheus.io/download/) \(make sure you select the appropriate processor architecture\), and use wget to download it and tar to unpack the archive:

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

Next, we need to move the binaries, set ownership, and move config files to appropriate locations:

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

`/etc/prometheus` is used for configuration, and `/var/lib/prometheus` for data.

Let’s set up Prometheus to run as a system service. Do**:**

```cpp
sudo nano /etc/systemd/system/prometheus.service
```

\(or open that file in the text editor of your choice\), and enter the following configuration:

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

Save the file. Now, we can run Prometheus as a system service:

```cpp
sudo systemctl daemon-reload
```

```cpp
sudo systemctl start prometheus
```

```cpp
sudo systemctl enable prometheus
```

Prometheus should now be running. To make sure, we can check with:

```cpp
systemctl status prometheus
```

which should produce something like:

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

You can also check Prometheus web interface, available on `http://your-node-host-ip:9090/`

{% hint style="warning" %}
You may need to do `sudo ufw allow 9090/tcp` if the firewall is on**.**
{% endhint %}

## Install Grafana

To set up Grafana project repositories with Ubuntu:

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

To install Grafana:

```cpp
sudo apt-get update
```

```cpp
sudo apt-get install grafana
```

To configure it as a service:

```cpp
sudo systemctl daemon-reload
```

```cpp
sudo systemctl start grafana-server
```

```cpp
sudo systemctl enable grafana-server.service
```

To make sure it’s running properly:

```text
sudo systemctl status grafana-server
```

which should show grafana as `active`. Grafana should now be available at `http://your-node-host-ip:3000/`

{% hint style="warning" %}
You may need to do `sudo ufw allow 3000/tcp` if the firewall is on**.**
{% endhint %}

Log in with username/password admin/admin and set up a new, secure password. Now we need to connect Grafana to our data source, Prometheus.

On Grafana’s web interface:

* Go to Configuration on the left-side menu and select Data Sources.
* Click Add Data Source
* Select Prometheus.
* In the form, enter the name \(Prometheus will do\), and `http://localhost:9090` as the URL.
* Click `Save & Test`
* Check for “Data source is working” green message.

## Set up node\_exporter

In addition to metrics from AvalancheGo, let’s set up up monitoring of the machine itself, so we can check CPU, memory, network and disk usage and be aware of any anomalies. For that, we will use node\_exporter, a Prometheus plugin.

Get the latest version with:

```text
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest | grep browser_download_url | grep linux-amd64 |  cut -d '"' -f 4 | wget -qi -
```

change `linux-amd64` if you have a different architecture \(RaspberryPi is `linux-arm64`, for example\). Untar and move the executable:

```cpp
tar xvf node_exporter-1.0.1.linux-amd64.tar.gz
```

```cpp
sudo mv node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin
```

```cpp
node_exporter --version
```

Then we add node\_exporter as a service. Do:

```cpp
sudo nano /etc/systemd/system/node_exporter.service
```

\(or open that file in the text editor of your choice\) and populate it with:

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

This configures node\_exporter to collect various data we might find interesting. Start the service, and enable it on boot:

```cpp
sudo systemctl start node_exporter
```

```cpp
sudo systemctl enable node_exporter
```

```cpp
sudo systemctl status node_exporter
```

Now, we’re ready to tie it all together.

## Configure AvalancheGo and node\_exporter Prometheus jobs

Make sure that your AvalancheGo node is running with appropriate [command line arguments](../../references/command-line-interface.md). The metrics API must be enabled \(by default, it is\). If you use CLI argument `--http-host` to make API calls from outside of the host machine, make note of the address at which APIs listen.

We now need to define an appropriate Prometheus job. Let’s edit Prometheus configuration:

Do :

```cpp
sudo nano /etc/prometheus/prometheus.yml
```

\(or open that file in the text editor of your choice\) and append to the end:

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

**Indentation is important**. Make sure `-job_name` is aligned with existing `-job_name entry`, and other lines are also indented properly. Make sure you use the correct host IP, or `localhost`, depending on how your node is configured.

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
eyJoaXN0b3J5IjpbLTQ0NTU4MTI4OF19
-->