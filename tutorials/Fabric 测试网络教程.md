# [Fabric 测试网络教程](https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html)
#区块链 #Fabric #教程

## 使用Fabric测试网络
在你下载了Hyperledger Fabric Docker镜像和样本后，你可以通过使用fabric-samples资源库中提供的脚本来部署一个测试网络。该测试网络是为通过在你的本地机器上运行节点来学习Fabric而提供的。开发人员可以使用该网络来测试他们的智能合约和应用程序。该网络只是作为教育和测试的工具，而不是作为如何建立一个网络的模型。一般来说，不鼓励对脚本进行修改，这可能会破坏网络。它是基于一个有限的配置，不应该被用作部署生产网络的模板。
- 它包括两个对等组织和一个订购组织。
- 为了简单起见，配置了一个单节点的Raft订购服务。
- 为了减少复杂性，没有部署TLS证书机构（CA）。所有的证书都是由根CA颁发的。
- 该样本网络用Docker Compose部署了一个Fabric网络。因为节点在Docker Compose网络中是隔离的，所以测试网络没有被配置为连接到其他正在运行的Fabric节点。

要了解如何在生产中使用Fabric，[请参阅部署生产网络](https://hyperledger-fabric.readthedocs.io/en/release-2.2/deployment_guide_overview.html)。

注意：这些说明已经过验证，可以在最新的稳定Docker镜像和所提供的tar文件中预编译的安装工具中使用。如果你用当前主分支的镜像或工具运行这些命令，你有可能会遇到错误。

##在你开始之前
在运行测试网络之前，你需要克隆 `fabric-samples` 仓库并下载 Fabric 镜像。

重要提示：本教程与Fabric测试网络样本v2.2.x兼容。在安装好先决条件后，你必须运行以下命令，克隆所需版本的hyperledger/fabric样本库，并签出正确的版本标签。该命令还将Hyperledger Fabric平台特定的二进制文件和该版本的配置文件安装到 `fabric-samples`的 `/bin` 和 `/config` 目录下。

```bash
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.2 1.4.9
```

## 构建测试网络
你可以在 `fabric-samples` 资源库的 `test-network` 目录中找到建立网络的脚本。使用以下命令导航到测试网络目录。

```bash
cd fabric-samples/test-network
```

在这个目录中，你可以找到一个有注释的脚本，network.sh，它可以在你的本地机器上使用Docker镜像建立一个Fabric网络。你可以运行./network.sh -h来打印脚本的帮助文本。

```bash
Usage:
  network.sh <Mode> [Flags]
    Modes:
      up - 启动Fabric orderer和peer节点。没有创建通道
      up createChannel - 启动fabric网络并创建通道
      createChannel - 网络创建后创建并加入一个通道
      deployCC - 部署一个节点到通道 (默认为 asset-transfer-basic)
      down - 关闭网络

    Flags:
    与 network.sh up, network.sh createChannel 一起使用:
    -ca <use CAs> -  使用Certificate Authorities生成网络加密材料
    -c <channel name> - 创建通道的名称（默认为 "mychannel"）
    -s <dbtype> - 要部署的对等状态数据库：goleveldb（默认）或couchdb
    -r <max retry> -  CLI在尝试一定次数后就会退出（默认为5次）。
    -d <delay> - CLI延迟一定的秒数(默认为3)
    -i <imagetag> - 要部署的Fabric的Docker镜像标签（默认为 "latest")
    -cai <ca_imagetag> - 要部署的Fabric CA的Docker图片标签（默认为 "latest")
    -verbose -  审慎模式

    与 network.sh deployCC 一起使用
    -c <channel name> - 部署chaincode的通道名称
    -ccn <name> - Chaincode名称。
    -ccl <language> - 要部署的混沌代码的编程语言：Go（默认）、java、javascript、typescript
    -ccv <version>  - Chaincode版本。1.0（默认），v2，3.x版，等等。
    -ccs <sequence>  - Chaincode定义序列。必须是一个整数，1（默认），2，3等。
    -ccp <path>  - Chaincode的文件路径
    -ccep <policy>  - （可选）使用签名策略语法的Chaincode认可策略。默认策略要求来自Org1和Org2的背书
    -cccg <collection-config>  - （可选）私人数据集合配置文件的文件路径
    -cci <fcn name>  - （可选）Chaincode初始化函数的名称。当提供一个函数时，将请求执行init并调用该函数
    -h - 打印此信息

 可能的模式和标志组合
   up -ca -r -d -s -i -cai -verbose
   up createChannel -ca -c -r -d -s -i -cai -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 举例:
   network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
   network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript
```

在 `test-network` 目录内，运行下面的命令来删除任何以前运行的容器或工件。

```bash
./network.sh down
```

然后，你可以通过发出以下命令来启动网络。如果你试图从另一个目录运行该脚本，你会遇到问题。

```bash
./network.sh up
```

这个命令创建了一个Fabric网络，由两个对等节点和一个订购节点组成。当你运行 `./network.sh up` 时，没有创建通道，我们可以在未来的步骤中创建。如果命令成功完成，你会看到正在创建的节点的日志。

```bash
Creating network "fabric_test" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating peer0.org2.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org1.example.com ... done
Creating cli                    ... done
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS                  PORTS                                            NAMES
1667543b5634   hyperledger/fabric-tools:latest     "/bin/bash"         1 second ago    Up Less than a second                                                    cli
b6b117c81c7f   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up 1 second             0.0.0.0:7051->7051/tcp                           peer0.org1.example.com
703ead770e05   hyperledger/fabric-orderer:latest   "orderer"           2 seconds ago   Up Less than a second   0.0.0.0:7050->7050/tcp, 0.0.0.0:7053->7053/tcp   orderer.example.com
718d43f5f312   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up 1 second             7051/tcp, 0.0.0.0:9051->9051/tcp                 peer0.org2.example.com
```

如果你没有得到这个结果，请跳到 "[故障排除](https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html#troubleshooting)"，以获得可能出了问题的帮助。默认情况下，网络使用cryptogen工具来建立网络。然而，你也可以[用Certificate Authorities来建立网络](https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html#bring-up-the-network-with-certificate-authorities)。

## 测试网络的组成部分
在你的测试网络部署完毕后，你可以花些时间来检查它的组件。运行下面的命令，列出所有正在你的机器上运行的Docker容器。你应该看到由 `network.sh` 脚本创建的三个节点。
```bash
docker ps -a
```

与Fabric网络互动的每个节点和用户都需要属于一个组织，以便参与网络。测试网络包括两个对等组织，Org1和Org2。它还包括一个维护网络订购服务的单一订购者组织。

[Peers](https://hyperledger-fabric.readthedocs.io/en/release-2.2/peers/peers.html) 是任何Fabric网络的基本组成部分。Peers 存储区块链账本，并在交易被提交到账本之前对其进行验证。对等体运行智能合约，其中包含用于管理区块链账本上资产的业务逻辑。

网络中的每个peer都需要属于一个组织。在测试网络中，每个组织各运行一个peer，即`peer0.org1.example.com` 和`peer0.org2.example.com` 。

每个Fabric网络还包括一个[订购服务(ordering service)](https://hyperledger-fabric.readthedocs.io/en/release-2.2/orderer/ordering_service.html)。虽然peers验证交易并将交易区块添加到区块链账本中，但他们并不决定交易的顺序或将其纳入新区块。在一个分布式网络中，peers可能彼此相距甚远，对交易的创建时间没有共同的看法。就交易的顺序达成共识是一个昂贵的过程，会给peers带来开销。

订购服务(ordering service)允许 peers 专注于验证交易并将其提交到分类帐。在订购节点从客户那里收到认可的交易后，他们就交易的顺序达成共识，然后将它们添加到区块中。然后，这些区块被分发到peer节点，这些节点将这些区块添加到区块链账本上。

该样本网络使用一个由订购者组织运营的单节点Raft订购服务。你可以看到在你的机器上运行的订购节点是orderer.example.com。虽然测试网络只使用单节点的订购服务，但生产网络会有多个订购节点，由一个或多个订购者组织运营。不同的订购节点将使用Raft共识算法，就整个网络的交易顺序达成协议。

## 创建一个通道
现在我们的机器上已经运行了 peer 和 orderer 节点，我们可以使用脚本为Org1和Org2之间的交易创建一个Fabric通道。通道是特定网络成员之间的私有通信层。通道只能由被邀请到通道的组织使用，而对网络的其他成员是不可见的。每个通道都有一个独立的区块链账本。被邀请的组织 "加入 "他们的同行到通道中，以存储通道账本并验证通道上的交易。

你可以使用 `network.sh` 脚本在Org1和Org2之间创建一个通道，并将他们的 peers 加入到通道中。运行下面的命令，创建一个默认名称为 `mychannel` 的通道。

```bash
./network.sh createChannel
```

如果命令成功，你可以看到在你的日志中打印出以下信息：

```shell
========= Channel successfully joined ===========
```


你也可以使用channel标志来创建一个具有自定义名称的频道。作为一个例子，下面的命令将创建一个名为 `channel1` 的频道。

```bash
./network.sh createChannel -c channel1
```

通道标志还允许你通过指定不同的通道名称来创建多个通道。在你创建了 `mychannel` 或 `channel1` 之后，你可以使用下面的命令来创建第二个名为 `channel2` 的通道。

```bash
./network.sh createChannel -c channel2
```

如果你想在一个步骤中启动网络并创建一个通道，你可以同时使用 `up` 和 `createChannel` 模式。

```bash
./network.sh up createChannel
```

## 在通道上启动一个 chaincode
在你创建了一个通道后，你可以开始使用[智能合约](https://hyperledger-fabric.readthedocs.io/en/release-2.2/smartcontract/smartcontract.html)与通道账本互动。智能合约包含管理区块链账本上资产的业务逻辑。网络成员运行的应用程序可以调用智能合约来创建账本上的资产，以及改变和转移这些资产。应用程序还可以查询智能合约以读取账本上的数据。

为了确保交易的有效性，使用智能合约创建的交易通常需要由多个组织签署，以提交给通道分类账。多重签名是Fabric信任模型的组成部分。要求一个交易有多个签名，可以防止通道上的一个组织篡改其同行的账本，或使用未同意的商业逻辑。为了签署一项交易，每个组织需要在他们的对等体上调用和执行智能合约，然后签署交易的输出。如果输出是一致的，并且已经被足够多的组织签署，那么该交易就可以被提交到分类帐。指定通道上需要执行智能合约的集合组织的政策被称为背书政策，它被作为 chaincode 定义的一部分为每个 chaincode 设置。

在Fabric中，智能合约是以被称为Chaincode的包的形式部署在网络上。一个Chaincode被安装在一个组织的peers身上，然后被部署到一个通道上，然后它可以被用来认可交易并与区块链账本互动。在Chaincode被部署到一个通道之前，该通道的成员需要就建立Chaincode治理的Chaincode定义达成一致。当所需数量的组织同意时，链码定义可以提交给通道，链码就可以使用了。

在你使用 `network.sh` 创建一个频道后，你可以使用以下命令在该频道上启动一个chaincode。

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

`deployCC` 子命令将在 `peer0.org1.example.com` 和 `peer0.org2.example.com` 上安装**asset-transfer (basic)** chaincode，然后将chaincode部署在用channel标志指定的channel上（如果没有指定channel，则是mychannel）。如果你是第一次部署chaincode，脚本将安装chaincode的依赖项。你可以使用语言标志-l来安装Go、typescript或javascript版本的chaincode。你可以在 `fabric-samples` 目录下的`asset-transfer-basic` 文件夹中找到asset-transfer（basic）chincode。这个文件夹包含的samples chincode是作为例子提供的，被教程用来突出Fabric的功能。



**`译者注：此处必须先安装完成golang环境，并建议使用代理：配置命令如下：`**

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

## 与网络互动
在你启动测试网络后，你可以使用 `peer CLI` 与你的网络互动。`peer CLI` 允许你调用已部署的智能合约，更新通道，或从CLI安装和部署新的智能合约。

确保你是在test-network目录下操作的。如果你按照说明[安装样例、可执行文件和Docker 镜像](https://hyperledger-fabric.readthedocs.io/en/release-2.2/install.html)，你可以在 `fabric-samples` 仓库的 `bin` 文件夹中找到 `peer` 的可执行文件。使用以下命令将这些二进制文件添加到你的CLI路径中：

```bash
export PATH=${PWD}/../bin:$PATH
```

你还需要设置 `FABRIC_CFG_PATH`，使其指向 ` fabric-samples`  资源库中的 `core.yaml` 文件。

````bash
export FABRIC_CFG_PATH=${PWD}/../config/
````

现在你可以设置环境变量，使你能以Org1的身份操作 `peer` 的CLI。

```bash
# Org1的环境变量

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

`CORE_PEER_TLS_ROOTCERT_FILE` 和 `CORE_PEER_MSPCONFIGPATH` 环境变量指向 `organizations` 文件夹中的Org1加密材料。

如果使用 `./network.sh deployCC -ccl go` 安装并启动asset-transfer (basic) chaincode，可以调用（Go）chaincode 的 `InitLedger` 函数，将初始的资产列表放到账本上（如果使用typecript或javascript `./network.sh deployCC -ccl javascript` 为例，将调用各自 chaincode 的 `InitLedger` 函数）。

运行以下命令，用 assets 初始化账本。

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function": "InitLedger", "Args":[]}'
```

如果成功，你应该看到与下面类似的输出：

```shell
-> INFO 001 Chaincode invoke successful. result: status:200
```


现在你可以从CLI查询账本了。运行下面的命令来获取被添加到 channel 账本的 资产列表。

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

如果成功，你应该看到以下输出：

```json
[
  {"ID": "asset1", "color": "blue", "size": 5, "owner": "Tomoko", "appraisedValue": 300},
  {"ID": "asset2", "color": "red", "size": 5, "owner": "Brad", "appraisedValue": 400},
  {"ID": "asset3", "color": "green", "size": 10, "owner": "Jin Soo", "appraisedValue": 500},
  {"ID": "asset4", "color": "yellow", "size": 10, "owner": "Max", "appraisedValue": 600},
  {"ID": "asset5", "color": "black", "size": 15, "owner": "Adriana", "appraisedValue": 700},
  {"ID": "asset6", "color": "white", "size": 15, "owner": "Michel", "appraisedValue": 800}
]
```

当网络成员想要转移或改变账本上的资产时，就会调用Chaincodes。使用下面的命令，通过调用asset-transfer (basic) chaincode来改变分类账上的资产所有者。

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

如果命令成功，你应该看到以下响应。

```go
2019-12-04 17:38:21.048 EST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

因为asset-transfer (basic) chaincode的背书策略要求交易由Org1和Org2签署，所以chaincode调用命令需要使用 `--peerAddresses` 标志来针对`peer0.org1.example.com` 和 `peer0.org2.example.com` 。由于网络启用了TLS，该命令还需要使用 `--tlsRootCertFiles` 标志引用每个peer 的TLS证书。

在我们调用Chaincode后，我们可以使用另一个查询来查看调用如何改变区块链账本上的资产。由于我们已经查询了Org1 peer，我们可以利用这个机会查询运行在Org2 peer上的 chaincode。设置以下环境变量，作为Org2的操作：

```bash
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

现在你可以查询在 `peer0.org2.example.com` 上运行的asset-transfer (basic) chaincode·

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

结果将显示，`"asset6"` 被转移给了 `“Christopher”`。

```bash
{"ID":"asset6","color":"white","size":15,"owner":"Christopher","appraisedValue":800}
```

## 关闭网络
当你完成了测试网络的使用，你可以用以下命令关闭网络：

```bash
./network.sh down
```

该命令将停止并删除节点和Chaincode容器，删除组织加密材料，并从Docker注册表中删除Chaincode镜像。该命令还删除了以前运行channel产生的持久化数据以及docker的持久化数据，如果你遇到任何问题，可以再次运行 `./network.sh up` 了。

## 接下来的步骤
现在你已经使用测试网络在你的本地机器上部署了 `Hyperledger Fabric` ，你可以使用教程开始开发自己的解决方案：

- 使用[将智能合约部署到通道](https://hyperledger-fabric.readthedocs.io/en/release-2.2/deploy_chaincode.html)的教程，学习如何将自己的智能合约部署到测试网络。
- 访问[编写你的第一个应用程序](https://hyperledger-fabric.readthedocs.io/en/release-2.2/write_first_app.html)的教程，学习如何使用Fabric SDKs提供的API，从你的客户端应用程序调用智能合约。
- 如果你已经准备好将一个更复杂的智能合约部署到网络上，请跟随[商业票据教程](https://hyperledger-fabric.readthedocs.io/en/release-2.2/tutorial/commercial_paper.html)，探索一个两个组织使用区块链网络交易商业票据的用例。

你可以在教程页面上找到[Fabric教程的完整列表](https://hyperledger-fabric.readthedocs.io/en/release-2.2/tutorials.html)。



## 用认证机构（Certificate Authorities）启动网络
Hyperledger Fabric使用公钥基础设施（PKI）来验证所有网络参与者的行为。每个节点、网络管理员和提交交易的用户都需要有一个公共证书和私钥来验证其身份。这些身份需要有一个有效的信任根，确定这些证书是由网络成员的一个组织颁发的。`network.sh` 脚本在创建peer节点和orderer节点之前，会创建所有部署和运行网络所需的加密材料。

默认情况下，该脚本使用 cryptogen 工具来创建证书和密钥。该工具是为开发和测试而提供的，它可以为具有有效信任根的织物组织快速创建所需的加密材料。当你运行 `./network.sh up` 时，你可以看到 cryptogen 工具为 Org1、Org2 和 Orderer Org 创建证书和密钥。

```bash
creating Org1, Org2, and ordering service organization with crypto from 'cryptogen'

/Usr/fabric-samples/test-network/../bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################

##########################################################
############ Create Org1 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
+ set +x
##########################################################
############ Create Org2 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
+ set +x
##########################################################
############ Create Orderer Org Identities ###############
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
+ set +x
```

事实上，测试网络脚本也提供了使用证书授权机构（CA）启动网络的选项。在一个生产网络中，每个组织运行一个CA（或多个中间CA），创建属于他们组织的身份。由该组织运行的CA所创建的所有身份都共享同一个信任根。虽然它比使用密码源需要更多的时间，但使用CA建立测试网络提供了一个关于如何在生产中部署网络的体验。部署CA还允许你用Fabric SDKs注册客户身份，并为你的应用程序创建一个证书和私钥。

如果你想使用Fabric CA来建立一个网络，首先运行以下命令来关闭任何正在运行的网络。

```bash
./network.sh down
```

然后你可以用CA参数启动网络：

```bash
./network.sh up -ca
```

在你执行操作后，你可以看到脚本启动了三个CA，网络中的每个组织都有一个。

```bash
##########################################################
##### Generate certificates using Fabric CA's ############
##########################################################
Creating network "net_default" with the default driver
Creating ca_org2    ... done
Creating ca_org1    ... done
Creating ca_orderer ... done
```

值得花时间检查CA部署后由 `./network.sh` 脚本产生的日志。测试网络使用Fabric CA客户端向每个组织的CA注册节点和用户身份。然后，该脚本使用 enroll 命令为每个身份生成一个 MSP 文件夹。MSP文件夹包含了每个身份的证书和私钥，并建立了该身份在运营CA的组织中的角色和成员资格。你可以使用下面的命令来检查Org1管理用户的MSP文件夹。

```bash
tree organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
```

该命令将显示出MSP的文件夹结构和配置文件：

```bash
organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
└── msp
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    │   └── localhost-7054-ca-org1.pem
    ├── config.yaml
    ├── keystore
    │   └── 58e81e6f1ee8930df46841bf88c22a08ae53c1332319854608539ee78ed2fd65_sk
    ├── signcerts
    │   └── cert.pem
    └── user
```

你可以在 `signcerts` 文件夹中找到管理用户的证书，在 `keystore` 文件夹中找到私钥。要了解更多关于MSP的信息，请参见[Membership Service Provider](https://hyperledger-fabric.readthedocs.io/en/release-2.2/membership/membership.html)概念主题。

cryptogen 和 Fabric CA 都在 `organizations` 文件夹中为每个组织生成加密材料。你可以在 `organizations/fabric-ca` 目录下的 `registerEnroll.sh` 脚本中找到用于设置网络的命令。要了解更多关于如何使用Fabric CA来部署Fabric网络，请访问[Fabric CA操作指南](https://hyperledger-fabric-ca.readthedocs.io/en/latest/operations_guide.html)。你可以通过访问[身份](https://hyperledger-fabric.readthedocs.io/en/release-2.2/identity/identity.html)和[成员](https://hyperledger-fabric.readthedocs.io/en/release-2.2/membership/membership.html)概念主题了解更多关于Fabric如何使用PKI的信息。

## 幕后发生了什么？
如果你有兴趣了解更多关于样例网络的信息，你可以研究 `test-network` 目录中的文件和脚本。下面的步骤提供了一个向导，介绍了当你发出 `./network.sh up` 命令时发生的事情。

- `./network.sh` 为两个 peer 组织和 orderer 组织创建证书和密钥。默认情况下，该脚本使用位于 `organizations/cryptogen` 文件夹中的配置文件使用 cryptogen 工具。如果你使用 `-ca` 标志来创建证书授权，该脚本会使用位于 `organizations/fabric-ca` 文件夹中的 Fabric CA 服务器配置文件和 `registerEnroll.sh` 脚本。cryptogen 和 Fabric CA 都在 `organizations` 文件夹中为所有三个组织创建加密材料和 MSP 文件夹。
- 一旦组织的密码材料被生成，`network.sh` 就可以启动网络的节点。该脚本使用 `docker` 文件夹中的 `docker-compos-test-net.yaml` 文件来创建peer和orderer节点。`docker` 文件夹还包含 `docker-compos-e2e.yaml` 文件，该文件将网络的节点和三个Fabric CA一起提出来。这个文件是用来运行Fabric SDK的端到端测试的。关于运行这些测试的细节，请参考[Node SDK repo](https://github.com/hyperledger/fabric-sdk-node)。
- 如果你使用 `createChannel` 子命令，`./network.sh` 运行 `scripts` 文件夹中的 `createChannel.sh` 脚本，使用提供的通道名称创建一个通道。该脚本使用configtxgen工具，根据 `configtx/configtx.yaml` 文件中的 `TwoOrgsApplicationGenesis` 通道配置文件创建通道创世块。创建通道后，脚本使用 `peer cli` 加入 `peer0.org1.example.com` 和 `peer0.org2.example.com` 到通道中，并使这两个peer成为锚点peer。
- 如果你发出 `deployCC` 命令，`./network.sh` 运行 `deployCC.sh` 脚本，在两个 peers 上安装 **asset-transfer (basic)** chaincode，然后在通道上定义chaincode。一旦chaincode定义被提交到通道上，`peer cli` 使用 `Init` 初始化链码，并调用chaincode将初始数据放到账本上。

## 疑难解答
如果你在使用本教程时遇到任何问题，请查看以下内容。

- 重启总是被优先考虑的操作。你可以使用下面的命令来删除以前运行的人工制品、加密材料、容器、卷和 chaincode 图像。

  ```bash
  ./network.sh down
  ```

  

  如果你不删除旧的容器、镜像和卷，你会看到 **错误**。

- 如果你看到Docker错误，首先检查你的Docker版本（先决条件），然后尝试重新启动Docker进程。Docker的问题往往不能立即识别。例如，你可能看到的错误是由于你的节点无法访问安装在容器内的加密材料而造成的。

  如果问题持续存在，你可以删除你的镜像并从头开始。

  ```bash
  docker rm -f $(docker ps -aq)
  docker rmi -f $(docker images -q)
  ```

- 如果你在macOS上运行Docker Desktop，并在安装chaincode时遇到以下错误。

  ```bash
  Error: chaincode install failed with status: 500 - failed to invoke backing implementation of 'InstallChaincode': could not build chaincode: docker build failed: docker image inspection failed: Get "http://unix.sock/images/dev-peer0.org1.example.com-basic_1.0-4ec191e793b27e953ff2ede5a8bcc63152cecb1e4c3f301a26e22692c61967ad-42f57faac8360472e47cbbbf3940e81bba83439702d085878d148089a1b213ca/json": dial unix /host/var/run/docker.sock: connect: no such file or directory
  Chaincode installation on peer0.org1 has failed(在peer0.org1上安装Chaincode失败了)
  Deploying chaincode failed(部署Chaincode失败)
  ```

  这个问题是由较新版本的Docker Desktop for macOS引起的。要解决这个问题，在Docker Desktop偏好中，取消勾选使用gRPC FUSE进行文件共享，以使用传统的osxfs文件共享，然后点击应用和重启。

- 如果你在创建、批准、提交、调用或查询命令中看到错误，请确保你已经正确更新了通道名称和链码名称。在提供的样本命令中，有占位符的数值。

- 如果你看到下面的错误。

  ```bash
  Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
  ```

  

  你可能有以前运行的 chaincode 图像（如 `dev-peer1.org2.example.com-asset-transfer-1.0` 或 `dev-peer0.org1.example.com-asset-transfer-1.0` ）。删除它们，然后再试一次。

  ```bash
  docker rmi -f $(docker images | grep dev-peer[0-9] | awk '{print $3}' )
  ```

- 如果你看到下面的错误。

  ```bash
  [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
  panic: Error reading configuration: Unsupported Config Type ""
  ```

  那么你没有正确设置 `FABRIC_CFG_PATH` 环境变量。`configtxgen` 工具需要这个变量，以便找到 `configtx.yaml` 。回去执行 `export FABRIC_CFG_PATH=$PWD/configtx/configtx.yaml` ，然后重新创建你的通道构件。

- 如果你看到一个错误，说明你仍然有 "活跃的端点"，那么就删减你的Docker网络。这将擦除你以前的网络，并以一个全新的环境开始。

  ```bash
  docker network prune
  ```

  你会看到以下信息。

  ```bash
  WARNING! This will remove all networks not used by at least one container.
  Are you sure you want to continue? [y/N]
  ```

  选择 `y` 。

- 如果你看到一个类似于下面的错误:

  ```bash
  /bin/bash: ./scripts/createChannel.sh: /bin/bash^M: bad interpreter: No such file or directory
  ```

  确保有问题的文件（本例中的**createChannel.sh**）是以Unix格式编码的。这很可能是由于在你的Git配置中没有将 `core.autocrlf` 设置为 `false` 造成的（见[Windows额外功能](https://hyperledger-fabric.readthedocs.io/en/release-2.2/prereqs.html#windows-extras)）。有几种方法可以解决这个问题。例如，如果你能使用vim编辑器，打开文件:

  ```bash
  vim ./fabric-samples/test-network/scripts/createChannel.sh
  ```

  然后通过执行以下vim命令改变其格式:

  ```bash
  :set ff=unix
  ```

如果你遇到其他错误，请在[Hyperledger Rocket Chat](https://chat.hyperledger.org/home)的**fabric-questions**频道或[StackOverflow](https://stackoverflow.com/questions/tagged/hyperledger-fabric)上分享你的日志。
