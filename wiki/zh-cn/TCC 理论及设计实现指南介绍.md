---
title: TCC 理论及设计实现指南介绍
author: zhangthen
date: 2019/03/26
keywords: fescar、分布式事务、TCC、roadmap
---

# TCC 理论及设计实现指南介绍

Fescar 0.4.0 版本发布了 TCC 模式，由蚂蚁金服团队贡献，欢迎大家试用，<br />Sample 地址：[https://github.com/fescar-group/fescar-samples/tree/master/tcc](https://github.com/fescar-group/fescar-samples/tree/master/tcc)，<br />文末也提供了项目后续的 Roadmap，欢迎关注。

<a name="f1d2fc6a"></a>
### 一、TCC 简介

在两阶段提交协议（2PC，Two Phase Commitment Protocol）中，资源管理器（RM, resource manager）需要提供“准备”、“提交”和“回滚” 3 个操作；而事务管理器（TM, transaction manager）分 2 阶段协调所有资源管理器，在第一阶段询问所有资源管理器“准备”是否成功，如果所有资源均“准备”成功则在第二阶段执行所有资源的“提交”操作，否则在第二阶段执行所有资源的“回滚”操作，保证所有资源的最终状态是一致的，要么全部提交要么全部回滚。

资源管理器有很多实现方式，其中 TCC（Try-Confirm-Cancel）是资源管理器的一种服务化的实现；TCC 是一种比较成熟的分布式事务解决方案，可用于解决跨数据库、跨服务业务操作的数据一致性问题；TCC 其 Try、Confirm、Cancel 3 个方法均由业务编码实现，故 TCC 可以被称为是服务化的资源管理器。

TCC 的 Try 操作作为一阶段，负责资源的检查和预留；Confirm 操作作为二阶段提交操作，执行真正的业务；Cancel 是二阶段回滚操作，执行预留资源的取消，使资源回到初始状态。

如下图所示，用户实现 TCC 服务之后，该 TCC 服务将作为分布式事务的其中一个资源，参与到整个分布式事务中；事务管理器分 2 阶段协调 TCC 服务，在第一阶段调用所有 TCC 服务的 Try 方法，在第二阶段执行所有 TCC 服务的 Confirm 或者 Cancel 方法；最终所有 TCC 服务要么全部都是提交的，要么全部都是回滚的。

![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/tcc.png)

<a name="48153343"></a>
### 二、TCC 设计

用户在接入 TCC 时，大部分工作都集中在如何实现 TCC 服务上，进过蚂蚁金服多年的 TCC 应用，总结如下主要的TCC 设计和实现主要事项：

<a name="4226dc7c"></a>
#### 1、**业务操作分两阶段完成**

接入 TCC 前，业务操作只需要一步就能完成，但是在接入 TCC 之后，需要考虑如何将其分成 2 阶段完成，把资源的检查和预留放在一阶段的 Try 操作中进行，把真正的业务操作的执行放在二阶段的 Confirm 操作中进行。

以下举例说明业务模式如何分成两阶段进行设计，举例场景：“账户A的余额中有 100 元，需要扣除其中 30 元”；

在接入 TCC 之前，用户编写 SQL：“update 账户表 set 余额 = 余额 -20 where 账户 = A”，便能一步完成扣款操作。

在接入 TCC 之后，就需要考虑如何将扣款操作分成 2 步完成：

* Try 操作：资源的检查和预留；

在扣款场景，Try 操作要做的事情就是先检查 A 账户余额是否足够，再冻结要扣款的 30 元（预留资源）；此阶段不会发生真正的扣款。

* Confirm 操作：执行真正业务的提交；

在扣款场景下，Confirm 阶段走的事情就是发生真正的扣款，把A账户中已经冻结的 30 元钱扣掉。

* Cancel 操作：预留资源的是否；

在扣款场景下，扣款取消，Cancel 操作执行的任务是释放 Try 操作冻结的 30 元钱，是 A 账户回到初始状态。

![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/tow_step.png)


<a name="bce861f1"></a>
#### 2、**并发控制**

用户在实现 TCC 时，应当考虑并发性问题，将锁的粒度降到最低，以最大限度的提高分布式事务的并发性。

以下还是以A账户扣款为例，“账户 A 上有 100 元，事务 T1 要扣除其中的 30 元，事务 T2 也要扣除 30 元，出现并发”。

在一阶段 Try 操作中，分布式事务 T1 和分布式事务 T2 分别冻结资金的那一部分资金，相互之间无干扰；这样在分布式事务的二阶段，无论 T1 是提交还是回滚，都不会对 T2 产生影响，这样 T1 和 T2 在同一笔业务数据上并行执行。

![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/conc.png) <br />

<a name="e945e352"></a>

#### 3、**允许空回滚**
**如下图所示，事务协调器在调用 TCC 服务的一阶段 Try 操作时，可能会出现因为丢包而导致的网络超时，此时事务管理器会触发二阶段回滚，调用 TCC 服务的 Cancel 操作，而 Cancel 操作调用未出现超时。

TCC 服务在未收到 Try 请求的情况下收到 Cancel 请求，这种场景被称为空回滚；空回滚在生产环境经常出现，用户在实现TCC服务时，应允许允许空回滚的执行，即收到空回滚时返回成功。

![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/empty_rollback.png)

<a name="e02f3ee9"></a>
#### 4、防悬挂控制

如下图所示，事务协调器在调用 TCC 服务的一阶段 Try 操作时，可能会出现因网络拥堵而导致的超时，此时事务管理器会触发二阶段回滚，调用 TCC 服务的 Cancel 操作，Cancel 调用未超时；在此之后，拥堵在网络上的一阶段 Try 数据包被 TCC 服务收到，出现了二阶段 Cancel 请求比一阶段 Try 请求先执行的情况，此 TCC 服务在执行晚到的 Try 之后，将永远不会再收到二阶段的 Confirm 或者 Cancel ，造成 TCC 服务悬挂。

用户在实现  TCC 服务时，要允许空回滚，但是要拒绝执行空回滚之后 Try 请求，要避免出现悬挂。

![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/susp.png)


<a name="5322a3d5"></a>
#### 5、幂等控制

无论是网络数据包重传，还是异常事务的补偿执行，都会导致 TCC 服务的 Try、Confirm 或者 Cancel 操作被重复执行；用户在实现 TCC 服务时，需要考虑幂等控制，即 Try、Confirm、Cancel 执行一次和执行多次的业务结果是一样的。<br />![image.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/miden.png)<br /><br />

<a name="Roadmap"></a>
### Roadmap

当前已经发布到 0.4.0 版本，后续我们会发布 0.5 ~ 1.0 版本，继续对 AT、TCC 模式进行功能完善和和丰富，并解决服务端高可用问题，在 1.0 版本之后，本开源产品将达到生产环境使用的标准。


![图片1.png](https://github.com/fescar-group/awesome-fescar/blob/master/img/roadmap.png)
