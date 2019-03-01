# 基于docker的以太坊集群的私有链开发环境

> 项目来源：https://github.com/HuaRongSAO/ethereum-testnet-docker

利用原始项目的内容无法顺利运行该项目，因此进行修改

## 目录

- 使用本地搭建
  - docker 安装
  - docker-compose 安装
  - 搭建eth私有链
  - 简单测试

## <span id="docker-install">1.docker</span>

安装地址: https://docs.docker.com/install/

如果觉得安装步骤太麻烦,官方提供一件安装脚本:  
Linux下可以运行:

```shell
  # 下载脚本
  curl -fsSL get.docker.com -o get-docker.sh
  # 运行脚本
  sudo sh get-docker.sh
```

安装成功后:

- 启动服务:
  - ubuntu: systemctl start docker
  - centos: service docker start
  - window: 直接打开windowd的客户端

```shell
# 运行 docker version
$ docker version

Client:
  Version:	18.03.0-ce
  API version:	1.37
  Go version:	go1.9.4
  Git commit:	0520e24
  Built:	Wed Mar 21 23:10:01 2018
  OS/Arch:	linux/amd64
  Experimental:	false
  Orchestrator:	swarm

Server:
 Engine:
    Version:	18.03.0-ce
    API version:	1.37 (minimum version 1.12)
    Go version:	go1.9.4
    Git commit:	0520e24
    Built:	Wed Mar 21 23:08:31 2018
    OS/Arch:	linux/amd64
    Experimental:	false

```

## 2.docker-compose

> Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。

文档: https://docs.docker.com/compose/

安装: Linux系统下:

```sh
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

- python环境可以使用pip安装:

```sh
sudo pip install -U docker-compose
```
其他系统请看
- (官网)[https://docs.docker.com/compose/install/]
- (Docker — 从入门到实践)[https://yeasy.gitbooks.io/docker_practice/content/compose/install.html]

运行:
```sh
$ docker-compose --version
docker-compose version 1.17.0, build ac53b73
```

## 3.搭建eth私有链
下载仓库: https://github.com/HuaRongSAO/ethereum-testnet-docker

```sh
git clone https://github.com/HuaRongSAO/ethereum-testnet-docker --depth 1
cd ethereum-testnet-docker 
```

文件目录结构:

```sh
.
├── docker-compose
├── docker-compose-standalone.yml .# 单独启动一个eth节点
├── docker-compose.yml # 单独启动一个eth节点， 一个初始节点， 一个网络状态
├── eth-netstats # 网络状态
│   ├── build.sh
│   └── Dockerfile
├── genesis # 初始化配置
│   ├── genesis.json
│   ├── keystore # 初始化 配置账户
│   │   ├── UTC--2018-03-20T06-02-05.635648305Z--d6e2f555878f29ca190b8ef6bf7de334e1c47e51
│   │   ├── UTC--2018-03-20T06-41-40.448119525Z--c94d9949d9f752b155639097a4829f729f25d660
│   │   ├── UTC--2018-03-20T06-42-00.884818336Z--bc5742401cbf13508e8ebe4de6c63620a979b03c
│   │   ├── UTC--2018-03-20T07-30-41.060693013Z--7f2cd86c26263c6108c4485a5d0276c1508838c1
│   │   └── UTC--2018-03-20T08-03-17.586134968Z--888d206a4f1b11f0e6d3dfd1204f27e937bb983d
│   └── password
├── get-docker.sh
├── geth-node  # 以太坊节点+api
│   ├── app.json
│   ├── build.sh
│   ├── Dockerfile
│   └── start.sh
├── install-compose.sh
├── readme.md
└── README.md

```

运行:

```sh
# 进入ethereum-testnet-docker
cd ethereum-testnet-docker
# 进入geth-node.
cd geth-node
# 创建节点镜像
sudo sh build.sh
#进入eth-netstats
cd ../eth-netstats
#创建网络状态镜像
sudo sh build.sh
# 在ethereum-testnet-docker目录下运行 启动以太坊
# 第一次运行可能会加载时间相对长,以后就快了
docker-compose up -d
# 查看运行状态
docker ps
# 运行状态如下
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                                                                                             NAMES
59380c23b730        ethereumdocker_eth         "/root/start.sh '--d…"   5 seconds ago       Up 3 seconds        8545-8546/tcp, 30303/tcp, 30303-30304/udp                                                         ethereumdocker_eth_1
fbe34c9110b1        ethereumdocker_bootstrap   "/root/start.sh '--d…"   5 seconds ago       Up 4 seconds        0.0.0.0:8545->8545/tcp, 0.0.0.0:30303->30303/tcp, 8546/tcp, 0.0.0.0:30303->30303/udp, 30304/udp   bootstrap
4d77ae7e987e        ethereumdocker_netstats    "npm start"              6 seconds ago       Up 4 seconds        0.0.0.0:3000->3000/tcp                                                                            netstats
```

折就是运行成功了,接下来你可以直接打开浏览器访问: http://127.0.0.1:3000   
界面仓库:https://github.com/cubedro/eth-netstats  
你就可以查看到运行状态了

能看到两个节点:

- bootstrap  # 初始化节点
- e69fe54d216d # 新加入的节点

接下来添加多个节点

```sh
# docker-compose scale 作用是启动多个eth服务
docker-compose scale eth=3
```

出现了三个新的node节点:

- e69fe54d216d
- 55f0f83728db
- 1770aec8a167

### 使用geth

使用docker exec进行与geth的交互

```sh
# 格式 docker exec -it ethereumdocker_eth_{第几个节点} geth attach ipc://root/.ethereum/devchain/geth.ipc
docker exec -it ethereumdocker_eth_1 geth attach ipc://root/.ethereum/devchain/geth.ipc
```

进入docker内部的geth就可以正常的使用geth的功能了.   
阅读geth文档: 
- [geth文档](https://www.ethereum.org/cli)

```sh
Welcome to the Geth JavaScript console!

instance: Geth/v1.8.3-unstable-faed47b3/linux-amd64/go1.10
coinbase: 0xd6e2f555878f29ca190b8ef6bf7de334e1c47e51
at block: 0 (Thu, 01 Jan 1970 00:00:00 UTC)
 datadir: /root/.ethereum/devchain
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> admin.nodeInfo.enode
"enode://3732c434c5330205b12ef83e31e59dd36b6d04fc8392770057a05b678483524c2c5649932bc3cc8bced54d94befbd0fda6daae988243489ce0faecfa7a5b0914@[::]:30303"
> # geth cli
```

### 开始挖矿

节点是默认不挖矿的需要手动开启挖矿:

```sh
docker exec -it ethereumdocker_eth_1 geth attach ipc://root/.ethereum/devchain/geth.ipc
# 开始挖矿
miner.start()
```

等带一段时间后,就能重127.0.0.1:3000看到区块了.

```sh
# eth 查看用户
eth.accounts
```

```sh
# stop 停止挖矿
miner.stop()
```
