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