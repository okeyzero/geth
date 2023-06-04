# ETH链节点搭建 保姆级详细教程
文档最后修改日期:2023.06.04

ETH由PoW转向PoS协议之后，每个节点就需要同时运行两个软件，执行客户端和共识客户端。本次目的是运行节点并不需要质押ETH。
其中 执行客户端有 （以前称为“eth1 客户端”，或仅称为“以太坊客户端”）
| 客户端                                          | 语言     | 操作系统：            | 网络                                      | 同步策略                       | 状态缓冲        |
| ----------------------------------------------- | -------- | --------------------- | ----------------------------------------- | ------------------------------ | --------------- |
| [Geth](https://geth.ethereum.org/)              | Go       | Linux、Windows、macOS | 主网、Sepolia、Görli、Ropsten、Rinkeby    | 快照、完全                     | Archive、Pruned |
| [Nethermind](http://nethermind.io/)             | C#、.NET | Linux、Windows、macOS | 主网、Sepolia、Görli、Ropsten、Rinkeby 等 | 快照（不提供服务）、快速、完全 | Archive、Pruned |
| [Besu](https://besu.hyperledger.org/en/stable/) | Java     | Linux、Windows、macOS | 主网、Sepolia、Görli、Ropsten、Rinkeby 等 | 快速、完全                     | Archive、Pruned |
| [Erigon](https://github.com/ledgerwatch/erigon) | Go       | Linux、Windows、macOS | 主网、Sepolia、Görli、Rinkeby、Ropsten 等 | 完全                           | Archive、Pruned |

其中共识客户端有 （以前称为“eth2”客户端）
| 客户端                                                        | 语言       | 操作系统：            | 网络                                                 |
| ------------------------------------------------------------- | ---------- | --------------------- | ---------------------------------------------------- |
| [Lighthouse](https://lighthouse.sigmaprime.io/)               | Rust       | Linux、Windows、macOS | 信标链、Goerli、Pyrmont、Sepolia、Ropsten 等         |
| [Lodestar](https://lodestar.chainsafe.io/)                    | TypeScript | Linux、Windows、macOS | 信标链、Goerli、Sepolia、Ropsten 等                  |
| [Nimbus](https://nimbus.team/)                                | Nim        | Linux、Windows、macOS | 信标链、Goerli、Sepolia、Ropsten 等                  |
| [Prysm](https://docs.prylabs.network/docs/getting-started/)   | Go         | Linux、Windows、macOS | 信标链、Gnosis、Goerli、Pyrmont、Sepolia、Ropsten 等 |
| [Teku](https://consensys.net/knowledge-base/ethereum-2/teku/) | Java       | Linux、Windows、macOS | 信标链、Gnosis、Goerli、Sepolia、Ropsten 等          |

本次搭建 选择 geth + Lighthouse 进行节点的搭建

### 一、服务器配置要求
 - [官方建议配置](https://ethereum.org/zh/developers/docs/nodes-and-clients/run-a-node/#getting-the-client)
```
系统：Linux& MacOS& Windows
CPU：4 核以上快速 CPU
内存：16GB 以上内存
带宽：25 MB/秒以上带宽
硬盘：1TB 以上高速固态硬盘
```

 - 本次搭建使用的服务器配置
```
系统：Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-71-generic x86_64)
CPU：16核
内存：128 GB DDR5 ECC
带宽：1 GBit/s
硬盘：2 x 1.92 TB NVMe SSD Datacenter Edition (Gen4)
区域：德国
```
----

 - geth官方文档：[https://geth.ethereum.org/docs/getting-started/hardware-requirements](https://geth.ethereum.org/docs/getting-started/hardware-requirements)
 - lighthouse官方文档：[https://lighthouse-book.sigmaprime.io/run_a_node.html](https://lighthouse-book.sigmaprime.io/run_a_node.html)
 

>  【注】以下命令 基于Ubuntu，不同linux版本 自行替换下命令  安装命令    Ubuntu：apt-get install ***    
> Centos： yum install *** -y 
> 如：安装 wget
>  Ubuntu： apt-get install wget    
>  Centos： yum install wget -y

 
----

### 二、服务器环境配置
1.  **更新Ubuntu/Centos软件包**

```bash
Ubuntu: sudo apt update && sudo apt dist-upgrade
Centos: yum -y upgrade
【注】若 apt 命令不行 就使用 apt-get 命令
```
2. **安装wget  git  vim  unzip 等**

```bash
Ubuntu:
apt install wget git screen gcc automake autoconf libtool make unzip liblz4-tool aria2 vim 
#其中 
#liblz4-tool 为解压用
#screen 为安装linux下的窗口管理器工具screen
 ```
## 三、安装执行客户端GETH
在根目录创建jiedian文件夹用来存放节点程序，并在同时在jiedian里边创建一个kuaizhao文件夹，下载的快照数据
创建文件夹
```bash
 cd /		#进入根目录
 mkdir eth		#创建jiedian及kuaizhao文件夹
 cd /eth		#进入jiedian文件夹
 ```
从[发布页面](https://geth.ethereum.org/downloads)下载预构建二进制文件

```bash
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.12.0-e501b3b0.tar.gz
# 解压压缩包
tar -zxvf geth-linux-amd64-1.12.0-e501b3b0.tar.gz
# 删除压缩包
rm -rf geth-linux-amd64-1.12.0-e501b3b0.tar.gz
# 设置可执行权限及重命名为geth
mv geth-linux-amd64-1.12.0-e501b3b0 geth
```

## 四、安装共识客户端Lighthouse
从[发布页面](https://github.com/sigp/lighthouse/releases)下载预构建二进制文件

```bash
cd /eth
wget https://github.com/sigp/lighthouse/releases/download/v4.2.0/lighthouse-v4.2.0-x86_64-unknown-linux-gnu.tar.gz
tar -zxvf lighthouse-v4.2.0-x86_64-unknown-linux-gnu.tar.gz
rm -rf lighthouse-v4.2.0-x86_64-unknown-linux-gnu.tar.gz
```
>修改 `/etc/profile`，在文件末尾添加了如下路径，目的是将  /eth/ 和 /eth/geth/ 加入环境变量，方便使用 geth 和 lighthouse
```bash
export PATH=/eth/:$PATH
export PATH=/eth/geth:$PATH
```
更新环境变量
```bash
source /etc/profile
```
> 使用 `geth version` 确认geth安装正确
> 使用 `lighthouse --version` 确认lighthouse安装正确

## 五、启动节点
1. **创建 JWT 秘密文件**
```bash
sudo mkdir -p /secrets
openssl rand -hex 32 | tr -d "\n" | sudo tee /secrets/jwt.hex
```
2. **启动执行客户端(执行节点)geth**
```bash
# 建议使用nohup 或者 开一个screen 来运行节点
geth  --cache 32768 --datadir /data/ethereum --http --http.addr 0.0.0.0   --http.api "eth,net,engine,web3" --ws --ws.addr 0.0.0.0  --ws.api "eth,net,engine,web3"  --txlookuplimit 0  --rpc.gascap 0  --rpc.txfeecap 0 --authrpc.addr 0.0.0.0  --authrpc.port 8551  --authrpc.vhosts 0.0.0.0 --authrpc.jwtsecret /secrets/jwt.hex --rpc.allow-unprotected-txs --maxpeers 2000
```
> 默认同步模式 是从快照同步
> 各个参数的具体含义: [https://geth.ethereum.org/docs/fundamentals/command-line-options](https://geth.ethereum.org/docs/fundamentals/command-line-options)

3. **启动安装共识客户端(信标节点)Lighthouse**
```bash
lighthouse bn --network mainnet --execution-endpoint http://127.0.0.1:8551 --executio
n-jwt /secrets/jwt.hex --checkpoint-sync-url https://sync-mainnet.beaconcha.in --disable-deposit-contract-sync --http
```
>`--execution-endpoint`：执行客户端(geth) API 的 URL。如果执行客户端(geth) 与默认端口在同一台计算机上运行，​​则为 http://localhost:8551 或 http://127.0.0.1:8551
>`--execution-jwt`：Lighthouse 和执行客户端(geth)共享的 JWT secret 文件的路径。这是一种强制性的身份验证形式，可确保 Lighthouse 有权控制执行引擎。
>`--checkpoint-sync-url`: Lighthouse 支持从最近完成的检查点快速同步。检查点同步是可选的；但是，我们强烈推荐它，因为它比从创世纪同步要快得多，同时仍提供相同的功能。检查点同步是使用以太坊社区提供的 公共端点: [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) 完成的。例如，在上面的命令中，我们使用的是 `https://sync-mainnet.beaconcha.in`。
> `--http`: 公开信标链的 HTTP 服务器。默认监听地址为http://localhost:5052. 信标节点需要 HTTP API 才能接受来自管理密钥的验证器客户端的连接。

4. **节点状态监听**

```scala
geth attach http://127.0.0.1:8545
或 
geth attach /data/ethereum/geth.ipc 
```

> eth.syncing	#查看当前区块情况
> net.peerCount	#查看当前连接节点数量
> eth.blockNumber #当前同步到区块高度

eth.syncing 结果 说明：
> currentBlock: 14290861, #当前同步到区块高度 
> highestBlock: 14297354, #主网当前高度
> knownStates:297473485, 
> pulledStates: 297473485, 
> startingBlock: 14270385
> 
> 若结果为	`false`	为同步完成


【注意】
> 这里的端口如果修改配置文件了，就填写配置文件的端口即可
> 同步本次同步大概3-4小时左右 在同步期间 可能会出现 eth.syncing 结果为false eth.blockNumber  结果为0 的情况 这是正常的  具体日志 可以去查看 执行客户端和共识客户端的日志
