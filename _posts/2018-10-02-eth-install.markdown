---
layout: post
title:  "linux下安装以太坊(ETH/ETC)节点及简单的交互!"
date:   2018-10-01 12:31:21 +0800
tags: linux eth etc ethereum
color: rgb(255,90,90)
cover: '../assets/ethereum.jpg'
subtitle: 'eth node install!'
---
使用以太坊进行转账等操作可以自己安装节点目前大约300G左右硬盘即可，可以离线签名交易，不依赖中心化交易所或者去中心化钱包，保证自己以太坊的安全，免去了使用冷钱包的复杂操作，此章介绍如何在ubuntu16.04上安装parity节点，以及使用geth console来和链交互，离线交易使用python，将在后面介绍：

- **parity的安装**
- **linux下守护程序supervisor安装**
- **配置supervisor配置文件**
- **配置parity配置文件**
- **启动ETH-parity**
- **geth简单和链交互**

----
#### 1.安装parity
可进入parity官网下载安装包，也可以直接在github上下载源码
[parity官网](https://www.parity.io/ )
[parity github](https://github.com/paritytech/parity)
linux下也可直接使用wget下载(例如parity1.10.4)：
```shell
wget http://d1h4xl4cr1h0mo.cloudfront.net/v1.10.4/x86_64-unknown-linux-gnu/parity_1.10.4_ubuntu_amd64.deb
```
然后安装
```shell
dpkg -i parity_1.10.4_ubuntu_amd64.deb
```
----
#### 2.linux下守护程序supervisor安装
安装supervisor需要安装**Python2.4以上**的Python环境，本教程安装**py3**

python环境安装：

- Ubuntu系统软件源中没有Python3.6的源，需手动添加(安装其他Python版本可跳过此步骤)：
```shell
sudo add-apt-repository ppa:jonathonf/python-3.6
```
- 更新软件源
```shell
sudo apt-get update
```
- 安装python
```shell
sudo apt-get install python3.X
```

安装supervisor：
```shell
sudo apt-get install supervisor
```
----

#### 3.配置supervisor配置文件
supervisor安装完成之后，在/lib/systemd/system/下会有supervisor.service服务，在/etc/supervisor/conf.d/下面可以自由配置parity启动的配置文件
操作如下：
```shell
cd /etc/supervisor/conf.d/
vim parity.conf
```
parity.conf如下：
```conf
[program:parity]
directory=/data/parity
command=parity --ui-no-validation --config=/data/parity/parity.toml --reserved-peers=/data/parity/peers.txt --jsonrpc-threads=14
autostart=true
```
directory为parity运行的主目录，数据全部存储在此目录下
command为parity运行的命令，主要参数可参考[parity启动命令](https://wiki.parity.io/Configuring-Parity-Ethereum)
autostart表示守护进程，程序意外挂掉之后自动重启

```shell
cd /lib/systemd/system/
vim supervisor.service
```
```conf
[Unit]                                                                                           
Description=Supervisor process control system for UNIX                                           
Documentation=http://supervisord.org
After=network.target

[Service]
ExecStart=/usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl -c /etc/supervisor/supervisord.conf $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```
此配置主要起到守护作用，详细介绍可百度

----

#### 4.配置parity配置文件
配置parity启动所需要的环境：
```shell
mkdir -p /data/parity
mkdir -p /var/log/parity
cd /data/parity/
vim parity.toml
vim peers.txt
```
上面**peers.txt**为对等节点的**peers_url**,配置一个网络完好的peers能**加快同步速度**，使节点更为**健壮**
配置文件parity.toml如下：
```toml
# This config should be placed in following path:
#   ~/.local/share/io.parity.ethereum/config.toml
[parity]
chain = "ropsten"
base_path = "/data/parity/.local/share/io.parity.ethereum"
#mode = "active"

[rpc]
interface = '0.0.0.0'
apis = ["web3","eth", "pubsub", "net", "parity", "parity_pubsub", "traces", "rpc","personal"]


[ipc]
disable = false
path = "/data/parity/.local/share/io.parity.ethereum/jsonrpc.ipc"
apis = ["web3","personal","eth", "pubsub", "net", "parity", "parity_pubsub", "parity_accounts", "traces", "rpc"]


[network]
# Parity will maintain at most 100 peers.
max_peers = 100


[ui]
disable = true
interface = '0.0.0.0'

[misc]
logging = "own_tx=trace"
log_file = "/var/log/parity/parity.log"
color = true
```
以上启动的是ropsten测试链，如需其他链可更改chain = "ropsten"，例如chain = "mainnet"即是eth正式链
其他参数参考[parity启动参数](https://wiki.parity.io/Configuring-Parity-Ethereum)

peers.txt配置实例：
```toml
enode://abe7a7144fe9f573b7772afe3d85926fa97cadd2baf115489e987aa5edc2c4569bf0b5e0b78018d42f8da503f5311912475cd12b4e4e2aaa2e8c548f8a6a08d3@18.196.168.151:30303
enode://c2a9279c4f3cef7a097270275d98f2e0a63c4f07114ec55c20a745eb3fcba1f6d0a583ee6cc367513148dbfcbc39d789861c4d57b773e993a9805db7005683be@172.31.46.189:30303
```
----

#### 5.配置启动parity服务
开启/关闭/重启 parity服务
```shell
cd /lib/systemd/system/
systemctl start supervisor.service
systemctl restart supervisor.service
systemctl stop supervisor.service
```
启动之后即可看见parity运行的log, 或者以下命令：
```shell
tailf /var/log/parity/parity.log
```

#### 6.geth交互
安装geth

1. 下载geth压缩包
```shell
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.8.8-2688dab4.tar.gz
```

2. 解压缩
```shell
tar -xf geth-linux-amd64-1.8.8-2688dab4.tar.gz
```

3.连接ipc
```shell
cd geth目录下bin文件夹
./geth attach ipc:/data/parity/.local/share/io.parity.ethereum/jsonrpc.ipc
```
以上即可开启geth console与链进行交互，可以打币收币等等。
下面介绍简单的交互命令，详细可参考[以太坊私有链Geth控制台操作教程](https://www.jianshu.com/p/9fa31e4cdf4d) ：
创建新账户：
```shell
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0xc8248c7ecbfd7c4104923275b99fafb308bbff92"
```
输入两遍密码后，生成账户地址。以同样的方式，可创建多个账户，查看账户：
```shell
>eth.accounts
```
查看账户余额:
```shell
>eth.getBalance(eth.accounts[0])
0
```
以上即是eth在linux下安装教程，后面会讲解如何通过python生成硬化衍生钱包地址及如何离线签名交易，以及连上节点广播交易等等。

Check out the [eth][eth] for more info on how to get the most out of eth. File all bugs/feature requests at [Adriel’s GitHub repo][adriel-gh]. If you have questions, you can ask them on [adriel-mail][adriel-mail].

[eth]: https://etherscan.io/
[adriel-gh]:   https://github.com/adrielliu
[adriel-mail]: adriel.liu@aliyun.com
