# 2.0版本新特性

## 群组架构

群组架构是FISCO BCOS 2.0众多新特性中的主线，创造灵感来源于人人都熟悉的群聊模式——群的建立非常灵活，几个人就可以快速拉个主题群进行交流。同一个人可以参与到自己感兴趣的多个群里，并行地收发信息。现有的群也可以继续增加成员。

采用群组架构的网络中，根据业务场景的不同，可存在多个不同的账本，区块链节点可以根据业务关系选择群组加入，参与到对应账本的数据共享和共识过程中。该架构的特点是：

- 各群组独立执行共识流程，由群组内参与者决定如何进行共识，一个群组内的共识不受其他群组影响，各群组拥有独立的账本，维护自己的交易事务和数据，使得各群组之间解除耦合独立运作，可以达成更好的隐私隔离；

- 机构的节点只需部署一次，通过群组设置即可参与到不同的多方协作业务中，或将一个业务按用户、时间等维度分到各群组，群组架构可快速地平行扩展，在扩大了业务规模同时，极大简化了运维复杂度，降低管理成本。

更多的群组介绍，请参考[群组架构设计文档](./design/architecture/group.md)和[群组使用教程](./tutorial/group_use_cases.md)

## 分布式存储

FISCO BCOS 2.0新增了对分布式数据存储的支持，节点可将数据存储在远端分布式系统中，克服了本地化数据存储的诸多限制。该方案有以下优点：

- 支持多种存储引擎，选用高可用的分布式存储系统，可以支持数据简便快速地扩容；
- 将计算和数据隔离，节点故障不会导致数据异常；
- 数据在远端存储，数据可以在更安全的隔离区存储，这在很多场景中非常有意义；
- 分布式存储不仅支持Key-Value形式，还支持SQL方式，使得业务开发更为简便；
- 世界状态的存储从原来的MPT存储结构转为分布式存储，避免了世界状态急剧膨胀导致性能下降的问题；
- 优化了数据存储的结构，更节约存储空间。

同时，2.0版本仍然兼容1.0版本的本地存储模式。更多关于存储介绍，请参考[分布式存储操作手册](./manual/distributed_storage.md)

## 并行计算模型

2.0版本中新增了合约交易的并行处理机制，进一步提升了合约的并发吞吐量。

1.0版本以及大部分业界传统区块链平台，交易是被打包成一个区块，在一个区块中交易顺序串行执行的。
2.0版本基于预编译合约，实现一套并行交易处理模型，基于这个模型可以自定义交易互斥变量。
在区块执行过程中，系统将会根据交易互斥变量自动构建交易依赖关系图——DAG，基于DAG并行执行交易，最好情况下性能可提升数倍（取决于CPU核数）。

更多并行计算模型的介绍，请参考并行交易的[设计文档](./design/parallel/dag.md)和[使用手册](./manual/transaction_parallel.md)。

## 预编译合约

FISCO BCOS 2.0提供预编译合约框架，支持采用C++编写合约，其优势是合约调用响应更快，运行速度更高，消耗资源更少，更易于并行计算，极大提升整个系统的效率。FISCO BCOS内置了多个系统级的合约，提供准入控制、权限管理、系统配置、CRUD式的数据存取等功能，这些功能天然集成在底层平台里，无需手动部署。

FISCO BCOS提供标准化接口和示例，帮助用户进行二次开发，便于用户编写高性能的业务合约，并方便地部署到FISCO BCOS里运行。预编译合约框架兼容EVM引擎，形成了“双引擎”架构，熟悉EVM引擎的用户可以选择将Solidity合约和预编译合约结合，在满足业务逻辑的同时获得巨大的效率提升。

另外，还有类似CRUD操作等也由预编译合约实现，更多预编译合约的介绍，请参考[预编译设计文档](./design/virtual_machine/precompiled.md)和[预编译合约开发文档](./manual/smart_contract.html#id2)

## CRUD合约

FISCO BCOS 2.0新增符合CRUD接口的合约接口规范，简化了将主流的面向SQL设计的商业应用迁移到区块链上的成本。其好处显而易见：
- 与传统业务开发模式类似，降低了合约开发学习成本；
- 合约只需关心核心逻辑，存储与计算分离，方便合约升级；
- CRUD底层逻辑基于预编译合约实现，数据存储采用分布式存储，效率更高；

同时，2.0版本仍然兼容1.0版本的合约，更多关于CRUD接口的介绍，请参考[使用CRUD接口](./manual/smart_contract.html#crud)。

## 控制台

FISCO BCOS 2.0新增控制台，作为FISCO BCOS 2.0的交互式客户端工具。

控制台安装简单便捷，简单配置后即可和链节点进行通信，拥有丰富的命令和良好的交互体验，用户可以通过控制台查询区块链状态、读取和修改配置、管理区块链节点、部署并调用合约。控制台给用户管理、开发、运维区块链带来了巨大的便利，降低了操作繁琐性和使用门槛。

相比于传统的nodejs等脚本工具，控制台安装简单、使用体验更好。详细请查看[控制台使用手册](./manual/console.md)。

## 虚拟机

2.0版本引入了最新的以太坊虚拟机版本，支持Solidity 0.5版本。同时，引入了EVMC扩展框架，支持扩展不同虚拟机引擎。
底层内部集成支持interpreter虚拟机，未来可扩展支持WASM/JIT等虚拟机。

更多关于虚拟机的介绍，请参考[虚拟机设计文档](./design/virtual_machine/index.html)

## 密钥管理服务

2.0版本对落盘加密进行了重塑升级，开启落盘加密功能时，依赖KeyManager服务进行密钥管理，安全性更强。

KeyManager在Github开源发布，节点与KeyManager的交互协议是开放的，支持机构设计实现符合自身密钥管理规范的KeyManager服务，比如采用硬件加密机技术。
该部分更详细的文档请参考[使用文档](./manual/storage_security.md)和[设计文档](./design/features/storage_security.md)

## 准入控制

2.0版本对准入机制进行了重塑升级，包括网络准入机制和群组准入机制，在不同维度对链和数据访问进行安全控制。

采用新的权限控制体系，基于表进行访问权限的设计，另外还支持CA黑名单机制，可以实现对作恶/故障节点的屏蔽。
详情请查看[准入机制设计文档](./design/security_control/index.html)

## 异步事件

2.0版本同时支持交易上链异步通知、区块上链异步通知以及自定义的AMOP消息通知等机制。

## 模块重塑

2.0版本对核心模块进行升级重塑，进行模块化的单元测试和端对端集成测试，支持自动化持续集成和持续部署。

