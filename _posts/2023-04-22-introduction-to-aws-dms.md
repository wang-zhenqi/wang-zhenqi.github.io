---
layout: post
title: "AWS 数据库迁移服务简介"
description: "本文介绍了 AWS 数据库迁移服务（DMS）的基础知识，通过本文，读者可以初步了解该服务的功能、限制、使用方法"
tags: [aws, dms, explanation]
---

# 背景介绍

数据迁移是一项在数据工程中经常会遇到的工作，是指在一个计算机存储系统中，选择、准备、提取并转换数据，最终将其永久地传输至另一个计算机存储系统的过程。通常会由于存储设备的升级或维护、应用程序迁移、灾难恢复以及数据中心迁移等原因而迁移数据。在迁移数据时，应尽可能地使其自动化运行，以避免由于人工失误而引发问题。另外，迁移后的数据校验以及遗留数据的清理也可归为数据迁移的任务。

而数据库迁移则是在数据迁移的基础上更具体了一步，迁移的主体是数据库。它可以将数据从一个或多个源数据库迁移到一个或多个目标数据库（见下图）。最终，使用源数据库的应用程序也应切换到目标数据库上。

![data-migration](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2022-11/data-migration-process-305ed6d3f4c94dadbf8d4f1449188673.png)

# AWS DMS 概览

AWS 数据库迁移服务（AWS Database Migration Service, DMS）是 AWS 的一项云服务，可以帮助用户在本地和 AWS 云端系统的组合间迁移数据，支持关系型数据库、数据仓库、NoSQL 数据库以及其他类型的数据存储。

DMS 具有如下特性：
1. 支持一次性的迁移工作，也可以依照源系统数据的更新方式持续地同步目标系统中的数据。
2. 支持同构数据库迁移和异构数据库迁移。即源数据库系统和目标数据库系统既可以是相同的，也可以是不同的。但要求**至少一端的数据库系统是运行在 AWS 的服务上的**。
3. 在迁移的过程中，源数据库系统仍可以保持可用
4. 自动管理迁移所需的软硬件的部署、管理以及监测，可以按需扩展或缩减资源。
5. 提供自动故障转移，主服务器发生故障，备份服务器则可接管运行，从而尽可能避免服务中断。
6. 按使用资源收费，即用即付。

# DMS 基本架构及工作流程

![dms-replication-process](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2022-11/dms-replication-process-29b5612ef0732244aef1082a2d963466.png)

如图所示，DMS 服务的主要组成部分是一个运行在 EC2 上的复制实例。它通过终端节点（Endpoint）一端连接到源数据库，另一端连接目标数据库，以此来指定数据从哪里来、到哪里去。然后它会调度一个复制任务来移动数据。

在迁移过程中，除了一些数据量较大的业务需要在磁盘中缓冲，大部分的数据都会利用内存来迁移。另外，源数据库的更新记录以及日志文件也会存储在磁盘中。

用户可以在目标数据库上定义具体的目标表，也可利用 DMS 自动创建目标表并关联主键。另外，AWS 还提供了 schema 转换工具（Schema Conversion Tool, SCT），以帮助用户以源表为依据，自动创建目标表、索引、视图、触发器等等数据库组件。

使用 DMS 的基本步骤是：
1. 在网络中搜寻源数据库\*
2. 自动将源数据库中的 schema 转换为可兼容目标数据库的形式\*
3. 创建复制实例（Replication Instance）
4. 创建源和目标节点
5. 创建一个或多个迁移任务（Replication Task）

前两步并非必须的，而且涉及到更为复杂的工具和服务，为了避免理解困难，本文将会在后面介绍再来介绍它们。

# DMS 组件介绍

DMS 中包含5个组件，恰好对应上面的5个基本步骤。我们先从最基本的3个组件开始介绍。

## 复制实例（Replication Instance）

复制实例实际上就是一个运行着复制任务的 EC2 实例，一个实例可以运行多个复制任务。多个复制任务可以将多个源数据库迁移至多个目标数据库。但因为数据迁移工作需要消耗大量的内存和 CPU 资源，用户需要根据实际的数据量以及应用场景选择合适的实例类型。下图就是一个基本的复制实例的架构：
![replication-instance](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2022-12/datarep-intro-rep-instance1-cfa61ed5d33897517c8ce7b220c07213.png)

## 终端节点（Endpoint）

DMS 利用终端节点来访问源数据库和目标数据库。每个终端节点都可以被多个复制任务共用。一般地，用户需要给终端节点提供以下信息：
* 节点类型：源或是目标
* 数据库引擎：例如是 Oracle 还是 PostgreSQL
* 服务器：DMS 可访问到的域名或是 IP 地址
* 端口号：数据库服务器的端口号
* 加密：是否启用 SSL 模式，即是否要在传输中对数据加密
* 凭据（Credential）：拥有足够的数据库访问权限的用户名和密码

每个终端节点都可以设置额外的连接属性，以达到控制复制行为的目的。例如设置记录日志的细节、文件大小等等参数。不同的数据库支持的连接属性都不一而同，具体信息可以查询 [*Sources for AWS DMS*](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.Sources.html) 以及 [*Targets for AWS DMS*](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.Targets.html)。

## 复制任务（Replication Task）

复制任务负责将数据从源节点复制到目标节点。创建复制任务是正式开始进行迁移之前的最后一步，需要配置以下信息：
* 复制实例：指定由哪一个复制实例来执行此任务
* 源节点和目标节点
* 迁移模式：
	* 全加载（Full Load）：该选项仅将数据从源数据库迁移到目标数据库，在这个过程中会按需要创建数据表。这一选项需要中断源数据库的事务，因此只有在能接受宕机的情况下才建议采用这种方式。
	* 全加载 + 变化数据捕获（Change Data Capture, CDC）：迁移现有数据并复制正在进行的更改。这种模式下，可以在捕获变化数据同时完成全部数据的加载，然后再处理这些发生了改变的数据，最终使目标数据库与源数据库达到同步。这样就可以无宕机地进行数据迁移。
	* 仅限 CDC：仅复制数据更改。有些情况下，可能使用其他方式进行全加载会更有效——例如在同构迁移中，也许数据库自带的批量加载方式会比 DMS 更快。这时就可以用这种模式仅仅捕获当迁移开始进行后发生的数据变化。
* 目标表准备模式：定义了当迁移开始时目标表的行为
	* 什么都不做：如果目标表在迁移前就已建好，就可以选择这个模式。
	* 删除目标表：将目标表删除再重建，此方法确保了在迁移开始时目标表为空。但要注意 DMS 仅创建表、主键以及（在某些情况下）唯一索引，而不会创建二级索引、非主键约束或列数据默认值。另外，在使用此模式时，可能需要对目标数据库执行某些配置，否则可能会因目标数据库的限制而无法重建数据表。
	* 清空目标表（Truncate）：适用于“全加载”以及“全加载+CDC”的迁移模式。同样可以确保迁移开始时目标表为空，如果目标表不存在，DMS 会自动创建。因此需要提前创建好数据表的 schema。
* LOB[^1] 模式：定义了在迁移时要如何处理 LOB 列
	* 不包含 LOB 列
	* 全包含：不论这一列的 LOB 有多大都要将它迁移，这种模式显然会拖慢迁移速度
	* 有限地包含：将 LOB 截断为参数“**Max LOB Size**”所设置的大小。这样会有效提高运行速度。
* 数据表映射：指定要迁移的数据库中的各个表及其 schema，这就意味着可以在迁移中对数据进行一定的转换和过滤。
* 数据转换（Transformation）
	* 更改 schema，表名及列名
	* 更改 tablespace 的名称（针对 Oracle 目标节点）
	* 定义目标表的主键和唯一索引
* 数据校验
* AWS CloudWatch 日志

下图展示了复制任务的基本架构。可以看到，数据的加载以及变化的捕获是复制任务的主要职责。在它会将捕获到的变化记录在 Change Cache 里，将任务执行的情况记录在 DMS Logs 里。
![replication-task](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2022-12/datarep-intro-rep-task1-dc8f14fd6556273fe8e4eb1ebd7f4a6d.png)

## 数据库发现（Database Discovery）

这个组件需要利用 DMS Fleet Advisor。它可以在多种数据库环境下（包括本地的和线上的）收集有关数据库服务器、数据库、schema 等等的信息，并将这些信息合成一个数据库编目（Inventory）供用户查看，以便用户分析数据迁移的策略，或者决定对哪些数据库进行监控。

## Schema 转换

用户可以利用这一组件来设计 schema 的转换过程，它由三部分组成：
1. Instance Profile：指定网络和安全设置
2. Data Provider：存储数据库连接凭证
3. Migration Project：包含了 Instance Profile、Data Provider 和迁移规则（Migration Rules）

上文已经提到，最后这两个组件并非必须，它们的功能可由三个基本组件实现。

# 一些细节的讨论

## 如何选择复制实例的类型？

从 DMS 的[文档](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReplicationInstance.Types.html)中可以看到，DMS 所支持的 EC2 实例类型有 T2, T3, C3, C4, R4 和 R5 等。具体每种类型的参数可以在文档中找到，这里不做赘述。总结一下，T2、T3 这类型号是通用型的，价格不贵，性能也一般，可用作简单的测试、验证等工作；C3、C4 这类型号是 CPU 增强版，主要可用来执行一些对算力有要求的工作；R4、R5 这类型号是 内存增强版，对与内存有要求的任务可以优先选择这类。

另外还需注意，DMS 在迁移数据时，是利用内存来存储中间结果的，所以大部分情况下，应该尽可能地满足内存的需要。尤其是在源数据库上有较高的 TPS (Transactions Per Second) 时，可能会需要大量的内存空间来记录数据的变化。

## 如何实现高可用（High Availability）以及故障转移（Failover）？

DMS 提供了多可用区域（Multi-Availability Zone）的支持。也就是将一个主复制实例部署在某个 AZ，再将一个备用复制实例部署在另一个 AZ。这一个主备的部署过程是由 DMS 自动完成的。主复制实例会与备用复制实例保持实时同步。如果主复制实例宕机了，那么备用复制实例就会自动接手主实例的工作。因为它们之间的状态总是保持一致的，因此备用实例的接入并不会造成太久的中断。示意图如下：

![dms-high-availability](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2022-12/dms-high-availability-ea9fc3daaca9419ac437ae02c0a4eb00.png)

## DMS 是如何实现“全加载 + CDC” 的？

在一个“全加载 + CDC”的复制任务里一共有三个步骤：
1. 将已有的数据迁移至目标数据库，即全加载
2. 应用已缓存的变化
3. 持续复制

在全加载的过程中，已有的数据会从源数据库被移动到目标数据库。任何在源数据表上产生的改动都会被缓存到复制实例上，只有当一张表开始进行全加载时，DMS 才会开始缓存它的变化。即此阶段的变化缓存是针对表的。

当一张表完成了全加载，DMS 就会立即开始应用已缓存的改动。在这一过程中，DMS 仍会以事务的形式记录各种改动。在这个阶段里，变化数据的捕获是基于数据库的，因为某一事务可能会更改多张表。如果在事务发生时，还有涉及到的表为完成全加载，那么这一事务也会暂存在磁盘上。

这样一来，DMS 一方面不断地应用已缓存的表的变化，另一方面也在持续记录库中的事务更改，最终会使得源和目标数据库达到一个平衡，也就是事务一致化。这时可以关闭应用程序，等待最后的一些事务应用到目标数据库上，最后重启应用程序并使它连接到新的数据库。

# DMS 的安全方案

1. 利用 AWS IAM（Identity and Access Management）策略来定义谁可以管理、访问 DMS 的资源。
2. DMS 可以使用 SSL（Secure Sockets Layer）协议来加密与终端节点之间的连接，以保证数据在传输的过程中是受保护的。
3. DMS 使用 AWS KMS（Key Management Service）的密钥来加密复制实例上的存储空间以及与终端节点的连接信息。
4. DMS 利用 VPC（Virtual Private Cloud）来作为整个迁移系统的访问控制机制，每个 VPC 都要关联一个安全组以控制出站、入站的各种规则。

以上就是关于 AWS 数据库迁移服务的一些基本的介绍。在实际使用中，需要通盘考虑不同的源、目标数据库的特性，以及业务需求，以选择最合适的实例类型、复制任务的模式等等。有时可能需要若干次试验才能找到合适的配置，在迁移完成后可能还需进行数据校验，或者在遇到问题时进行故障排查。这些更为细节的问题可留待实际遇到后再做总结。

[^1]: Large OBject (LOB): 在数据表的某一列中可能存放有巨大的二进制文件，这样的文件称为 LOB，这一列就是 LOB Column。