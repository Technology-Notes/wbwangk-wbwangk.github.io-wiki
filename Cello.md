## 概述
[[原文](https://cello.readthedocs.io/en/latest/#welcome-to-hyperledger-cello)]  
Hyperledger Cello是一个区块链模块工具包，也是Linux基金会托管的Hyperledger项目之一。Hyperledger Cello旨在为区块链生态系统提供按需“即服务”部署模型，以减少创建、管理和终止区块链所需的工作量。它在各种基础设施（例如，裸机、虚拟机和多个容器平台）之上高效且自动地提供多租户区块链服务。Hyperledger Cello最初由IBM提供，赞助商来自Soramitsu、华为和英特尔。

![典型情景](https://cello.readthedocs.io/en/latest/imgs/scenario.png)

Hyperledger Cello（HLC）是一种区块链配置和操作系统，可帮助人们以更有效的方式使用和管理区块链。

基于先进的区块链技术和现代PaaS工具，Cello提供以下主要功能：

- 管理区块链网络的生命周期，例如，`create/start/stop/delete/keep health`自动化。
- 支持定制的区块链网络配置，例如网络规模、共识类型。
- 支持多个底层基础设施，包括裸机、虚拟机、vSphere、本机[Docker](https://www.docker.com/)主机、swarm和[Kubernetes](https://kubernetes.io/)。更多支持还在开发中。
- 通过与现有工具（如[ElasticStack）](https://www.elastic.co/)集成，扩展了监控、日志记录、运行状况和分析功能等高级功能。

使用Cello，区块链开发人员可以：

- 从头开始快速构建区块链即服务（BaaS）平台。
- 立即提供可自定义的区块链，例如Hyperledger fabric v1.x网络。
- 检查系统状态并管理区块链，通过仪表板上传智能合约并测试等。
- 在裸机、虚拟机云（例如，虚拟机、vsphere云）、容器集群（例如，Docker、Swarm、Kubernetes）之上维护一个运行中的区块链网络池。

## 教程

### 基本概念

先看看[术语](https://cello.readthedocs.io/en/latest/terminology/)先了解基本概念。

### 安装一个Cello群集

按照[安装指南](https://cello.readthedocs.io/en/latest/setup/)启动一个Cello群集。

之后，操作员可以通过操作员仪表板与Cello进行交互。

默认情况下，操作员仪表板将侦听主节点上的`8080`端口，默认管理员帐户为`admin:pass`。

### 添加主机

第一次打开操作员仪表板时，将没有主机。有两种方法可以将更多主机添加到池中。

- 通过`Overview`页面：点击`Working Hosts`后的`+`按钮;
- 通过`Hosts`页面：点击右上角的`Add Host`按钮。

然后，您将看到一个跳出的对话框，根据所选类型输入设置信息。

![仪表板添加主机](https://cello.readthedocs.io/en/latest/imgs/tutorial_add_host.png)

假设要导入本机Docker服务器，您可能需要输入这些字段

- 名称：docker_host
- 守护进程URL :( `192.168.7.220:2375`用您的docker主机地址替换它）
- 容量（该主机上的最大链数）：5
- 日志记录级别：默认是DEBUG，你可以改变成`INFO`，`NOTICE`，`WARNING`，`ERROR`，`CRITICAL`
- 日志类型：默认为LOCAL
- 可以为群集请求调度：True或False。如果设置成True，它将安排一个链请求到该主机，这在维护主机时很有用
- 保持充满集群：True或False。如果设置为True，它将使用链将主机自动填充到其容量。

成功添加后，您可以在主机页面上找到`docker_host`，显示为0链和容量是5。

如果要创建vSphere类型主机，可以执行[vSphere类型主机创建指南](https://cello.readthedocs.io/en/latest/setup_worker_vsphere/)中的步骤。

如果要创建Kubernetes类型主机，可以执行[Kubernetes类型主机创建指南](https://cello.readthedocs.io/en/latest/setup_worker_kubernetes/)中的步骤。

### 创建一个链

现在我们在池中有了空闲主机，可以创建新链。

打开Active Chain页面，它现在应该是空的，单击右上角的`Add Chain`按钮，输入以下字段：

- 名称：test_chain

并选择主机`docker_host`。

![仪表板添加链](https://cello.readthedocs.io/en/latest/imgs/tutorial_add_chain.png)

单击“创建”按钮会将名称为`test_chain`的新链添加到池中。

然后，您可以在Active Chain页面上看到它。

### 启用自动模式

如果要手动创建多个链，将会很困难。Cello提供自动化方式来节省时间。

- 使用主机操作下拉菜单：Fillup按钮将使用链填充主机，直到其容量，而Clean按钮将清除主机中所有未使用的链。
- 使用“自动填充”复选框：在主机配置中，您可以找到一个`保持充满集群`复选框，该复选框将自动监视主机并使用链将其保持为满容量。

根据您的喜好尝试这些方法。

### 申请区块链

默认情况下，用户仪表板将侦听主节点上的`8081`端口，操作员可以使用默认`admin:pass`凭据登录。或者您可以转到“注册”以创建新帐户。

点击“立即申请”以申请一个新的区块链。填写“Name”字段作为链名称，并为链类型选择“Fabric”。然后将显示一个Fabric - Advance Config for configurations，继续选择默认配置，然后单击提交按钮以请求新的区块链。

![申请区块链](https://cello.readthedocs.io/en/latest/imgs/tutorial_apply_chain.png)

### 添加智能合约

默认情况下，有两个智能合约示例可用。您可以通过上传本地智能合约文件来添加新的智能合约。

[map example](https://github.com/hyperledger/cello/blob/master/user-dashboard/src/config-template/cc_code/examples/fabric/map/map.go)

这个链代码实现了一个存储在状态中的简单映射。

以下是可用的调用函数。

- put - 需要两个参数，一个键和一个值，并将它们存储在状态中
- remove  - 需要一个键并将其从状态中删除
- get - 需要一个参数，一个键，并返回一个值
- keys - 不需要参数，返回所有键

一个查询函数：

- query - 需要一个参数，一个键，并返回一个值

[fabric-example02 example](https://github.com/hyperledger/cello/blob/master/user-dashboard/src/config-template/cc_code/examples/fabric/chaincode_example02/chaincode_example02.go)

在此示例中，我们使用Init来配置账本上的变量的初始状态。该示例接受4个参数作为输入，并在账本上写入验证值。

- 第一个帐户名称
- 第一个帐户中的初始金额（整数）
- 第二个帐户名称
- 第二个帐户中的初始金额（整数）

可以使用以下操作。

- Invoke  - 需要两个参数，一个键和一个值，并将它们存储在状态中
- Query  - 需要一个密钥并将其从状态中删除

### 安装/部署智能合约

单击“fabric-chaincode_example02”智能合约的“...”，然后选择“安装”。在弹出窗口中，选择要安装智能合约的链。

![安装智能合约](https://cello.readthedocs.io/en/latest/imgs/tutorial_install_contract.png)

安装智能合约后，单击“实例化”以部署智能合约。在弹出窗口中，选择要实例化智能合约的链。然后单击“新参数”以添加四个参数，例如：“a”，“1000”，“b”，“2000”。

![部署智能合约](https://cello.readthedocs.io/en/latest/imgs/tutorial_deploy_contract.png)

### 调用/查询智能合约

在“调用”页面中，您可以按以下方式执行合约调用。

- 智能合约： `fabric-example02`
- 函数名称： `invoke`
- 参数：`a`，`b`，`100`
- 方法： `Invoke`

![调用智能合约](https://cello.readthedocs.io/en/latest/imgs/tutorial_invoke_contract.png)

然后我们查询当前的值`a`，现在应该是900

![查询智能合约](https://cello.readthedocs.io/en/latest/imgs/tutorial_query_contract.png)

### 操作员仪表板

如果您想了解更高级的操作技能，请继续阅读[操作员仪表板](https://cello.readthedocs.io/en/latest/dashboard_operator/)。

### 用户的仪表板

如果您想了解链和智能合约的更多用法，请继续访问[用户仪表板](https://cello.readthedocs.io/en/latest/dashboard_user/)。

## 术语

### 概述

在多个服务器上部署Cello，建议至少有1个Master + N（N> = 1）Worker。

- `Master`：运行Cello服务的管理节点。
- `Worker`：用于托管区块链的平台（例如，Docker，Swarm，Kubernetes，vSphere Cloud）。`Worker`由`Master`管理。
- `Host`：典型`worker`的资源将由一个唯一的平台管理。通常它可以是本地的Docker主机、Swarm集群、Kubernetes clsuter或其他裸机/虚拟/容器集群。
- `Chain`（`Cluster`）：区块链网络，包括一定数量的peer+order节点。例如，Hyperledger Fabric网络、Sawthooth Lake或Iroha链。

### Master

该`Master`会包含主要的Cello[服务](https://cello.readthedocs.io/en/latest/service_management/)。

这是整个Cello服务的控制面板，大部分管理工作都应该在这里处理。

`Master`将管理在`Workers`中运行的区块链网络。

### Worker

`Workers`由`Master`服务管理，并帮助托管区块链。

### 主机（Hosts)

主机是由同一资源控制器管理的一组资源，可以是本机Docker主机、Swarm群集、Kubernetes群集或当前的某些云。

通常主机有几个属性：

- `Name`：别名，为人阅读方便。
- `Daemon URL`：Docker / Swarm Access的URL。
- `Capacity`：主机最多可以拥有的链的数量。
- `Logging Level`：此主机上链的默认日志记录级别。
- `Logging Type`：如何处理这些日志消息。
- `Schedulable`：此主机上的链是否可以调度给用户。
- `Autofill`：始终使用链自动填充主机。

### 链

链通常是一个区块链集群，例如，Fabric网络。

一个链有几个属性：

- `Name`：为人准备的别名。
- `Host`：链所在的主机。
- `Size`：链具有的节点数量。
- `Consensus`：链采用什么样的共识，取决于区块链技术。