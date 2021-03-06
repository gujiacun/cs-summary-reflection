---
title: Consul和Raft介绍
categories:
- 复习整理
tags: [Java底层]
description: 介绍Consul与Raft以及Consul和其他常见软件的对比。
draft: true
---

* 目录
{:toc}

在本文中，我们展示了具有多个实例的高性能应用程序中的领导选举基础。我们演示了Consul的会话管理和KV存储功能如何帮助获得锁并选择领导者。

Consul和Raft介绍

Consul解决的问题多种多样，但是每个单独的功能已由许多不同的系统解决。尽管没有一个单一的系统可以提供Consul的所有功能，但是还有其他选项可以解决其中的一些问题。

我们将Consul与其他一些选项进行比较。在大多数情况下，Consul不会与任何其他系统互斥。

> quorum在本文被翻译成了法定人数，即达到共识所需要的最少要求

# Consul与ZooKeeper，doozerd等

ZooKeeper，doozerd和etcd的体系结构都相似。这三个系统的服务器节点都需要一定数量的节点才能运行（通常是简单多数，与之相似的是绝对多数）。它们具有强一致性，并公开了可通过应用程序中的客户端库用于构建复杂的分布式系统的各种原语。

Consul还可使用单个数据中心内的服务器节点。在每个数据中心，Consul服务器都需要一个法定人数（quorum）来运行并提供强一致性。然而，Consul有对多个数据中心的本地支持，以及一个功能更丰富的gossip系统，将服务器节点和客户端连接起来。

当提供K/V存储时，这些系统都具有大致相同的语义：读取具有强一致性，并且面对网络分区，为了一致性而牺牲了可用性。但是，当这些系统用于高级案例时，差异变得更加明显。

这些系统提供的语义对于构建服务发现系统很有吸引力，但是必须强调，使用者必须构建这些功能，这一点很重要。对于ZooKeeper等，仅提供原始的K/V存储，并要求应用程序开发人员构建自己的系统以提供服务发现。相反，Consul为服务发现提供了一个武断（opinionated）的框架，并消除了猜测和开发工作。客户端只需注册服务，然后使用DNS或HTTP接口执行服务发现。而其他系统需要本地解决方案。

一个引人注目的服务发现框架必须包含运行状况检查以及故障的可能性。如果该节点发生故障或服务崩溃，知道节点A提供Foo服务是没有用的。单纯的系统使用心跳，并使用定期更新和TTL。这些方案需要与节点数量成线性关系的工作，并将需要放在固定数量的服务器上。另外，故障检测窗口至少与TTL一样长。

ZooKeeper提供了临时节点，这些临时节点是在客户端断开连接时被删除的K/V条目。它们比心跳系统更复杂，但仍然存在固有的可拓展性问题，并增加了客户端的复杂性。所有客户端必须维持与ZooKeeper服务器的活跃连接并执行保活。另外，这需要“厚（thick）客户端”，这些客户端很难编写，并且经常带来调试挑战。

Consul使用非常不同的体系结构进行健康检查。Consul客户端不仅在服务器节点上运行，而且还在集群中的每个节点上运行。这些客户端是gossip池的一部分，该池具有多种功能，包括分布式运行状况检查。gossip协议实现了一个有效的故障检测器，该检测器可以扩展到任何规模的集群，而无需将工作集中在任何选定的服务器组上。客户端还允许在本地运行更丰富的健康检查集，而ZooKeeper临时节点是非常原始的存活检查。使用Consul，客户端可以检查Web服务器是否正在返回200状态码以便检测内存利用率，磁盘空间等（这里不清楚，原文是that memory utilization is not critical, that there is sufficient disk space, etc）。Consul客户端公开了一个简单的HTTP接口，并避免以与ZooKeeper相同的方式向客户端公开系统的复杂性。

Consul为服务发现，运行状况检查，K/V存储和多个数据中心提供一流的支持。为了支持除简单的K/V存储以外的所有功能，所有其他这些系统都需要在其顶部构建其他工具和类库。通过使用客户端节点，Consul提供了仅需要瘦（thin）客户端的简单API。此外，可以通过使用配置文件和DNS接口完全避免开发，从而完全避免使用API​​。

## Gossip算法

- 最终一致性
- 弱一致性

Gossip算法又被称为反熵（Anti-Entropy），熵是物理学上的一个概念，代表杂乱无章，而反熵就是在杂乱无章中寻求一致，这充分说明了Gossip的特点：在一个有界网络中，每个节点都随机地与其他节点通信，经过一番杂乱无章的通信，最终所有节点的状态都会达成一致。每个节点可能知道所有其他节点，也可能仅知道几个邻居节点，只要这些节可以通过网络连通，最终他们的状态都是一致的，当然这也是疫情传播的特点。
要注意到的一点是，即使有的节点因宕机而重启，有新节点加入，但经过一段时间后，这些节点的状态也会与其他节点达成一致，也就是说，Gossip天然具有分布式容错的优点。

Gossip是一个带冗余的容错算法，更进一步，Gossip是一个最终一致性算法。虽然无法保证在某个时刻所有节点状态一致，但可以保证在”最终“所有节点一致，”最终“是一个现实中存在，但理论上无法证明的时间点。
因为Gossip不要求节点知道所有其他节点，因此又具有去中心化的特点，节点之间完全对等，不需要任何的中心节点。实际上Gossip可以用于众多能接受“最终一致性”的领域：失败检测、路由同步、Pub/Sub、动态负载均衡。
但Gossip的缺点也很明显，冗余通信会对网路带宽、CPU资源造成很大的负载，而这些负载又受限于通信频率，该频率又影响着算法收敛的速度。

# Consul与Eureka

Eureka是一个服务发现工具。该体系结构主要是客户端/服务器，每个数据中心有一组Eureka服务器，通常每个可用性区域一个。通常，尤里卡（Eureka）的客户使用嵌入式SDK来注册和发现服务。对于未本地集成的客户端，将使用Ribbon等挎斗（sidecar）通过Eureka透明地发现服务。

Eureka使用尽力而为复制提供了服务的弱一致性视图。当客户端在服务器上注册时，该服务器将尝试复制到其他服务器，但不提供任何保证。服务注册的生存时间（TTL）较短，要求客户端与服务器进行心跳检测。不健康的服务或节点将停止心跳，导致它们超时并从注册表中删除。发现请求可以路由到任何服务，由于尽力而为的复制，它可以处理陈旧或丢失的数据。这种简化的模型可以简化集群管理和高可伸缩性。

Consul提供了一套超级功能，包括更丰富的运行状况检查，K/V存储和多数据中心感知。Consul要求每个数据中心中有一组服务器，以及每个客户端上的代理，类似于使用诸如Ribbon之类的sidecar。Consul代理（agent）允许大多数应用程序不知道Consul，它们通过配置文件执行服务注册，并通过DNS或负载均衡器sidecars进行发现。

Consul提供了一个强大的一致性保证，因为服务器使用Raft协议复制状态。Consul支持多种健康检查，包括TCP，HTTP，Nagios/Sensu兼容脚本或基于Eureka的TTL。客户端节点参与基于gossip的健康检查，该检查分散了健康检查的工作，与集中式心跳不同，后者成为可扩展性的一个挑战。发现请求被路由到选择的Consul领导者，这让他们能够在默认情况下具有强一致性。允许陈旧读取的客户端使任何服务器都能处理其请求，从而实现线性扩展，例如Eureka。

Consul的强一致性意味着可以将其用作领导人选举和集群协调的锁定服务。尤里卡（Eureka）没有提供类似的保证，通常需要对需要执行协调或具有更强一致性要求的服务运行ZooKeeper。

Consul提供了支持面向服务的体系结构所需的功能的工具包。这包括服务发现，还包括丰富的运行状况检查（health checking），锁定（locking），K/V，多数据中心联合，事件系统和ACLs。Consul和consul-template、envconsul之类的工具生态系统都试图最小化集成所需的应用程序更改，以避免需要通过SDK进行本机集成。 Eureka是更大的Netflix OSS套件的一部分，该套件期望应用程序相对同质并紧密集成。结果，Eureka仅解决了一部分问题，并期望与ZooKeeper等其他工具一起使用。

# 共识协议

Consul使用共识协议来提供一致性（由CAP定义）。共识协议基于["Raft: In search of an Understandable Consensus Algorithm"](https://raft.github.io/raft.pdf)。有关Raft的直观说明，请参见[The Secret Lives of Data](http://thesecretlivesofdata.com/raft)。

## Raft协议概述

Raft是基于Paxos的共识算法。与Paxos相比，Raft被设计为具有更少的状态和更简单，更易理解的算法。

讨论Raft时，需要了解一些关键术语：
- 日志 - Raft系统中的主要工作单元是日志条目。一致性问题可以分解为复制日志。日志是条目的有序序列。条目包括任何集群更改：添加节点，添加服务，新的键值对等。如果所有成员都同意条目及其顺序，则我们认为日志是一致的。
- FSM - 有限状态机。FSM是有限状态的集合，在它们之间有过渡。应用新日志时，允许FSM在状态之间转换。应用相同的日志序列必须导致相同的状态，这意味着行为必须是确定性的。
- 对等集 - 对等集是参与日志复制的所有成员的集合。出于Consul的目的，所有服务器节点都位于本地数据中心的对等集中。
- 法定人数 - 法定人数是来自对等集合的大多数成员：对于大小为N的集合，法定人数至少需要`(N/2)+1`个成员。例如，如果在对等集中有5个成员，我们将需要3个节点来形成法定人数。如果由于某些原因无法达到法定数量的节点，则集群将变得不可用，并且无法提交任何新日志。
- 提交的条目 - 将条目持久存储在法定数量的节点上时，该条目被视为已提交。提交条目后，即可应用它。
- 领导者 - 在任何给定时间，对等集都选择一个节点作为领导者。领导者负责摄取新的日志条目，复制到关注者，并管理何时将条目视为已提交。

Raft是一个复杂的协议，在此不做详细介绍（对于那些需要更全面了解的人，可以在[本文](https://raft.github.io/raft.pdf)中找到完整的规范）。但是，我们将尝试提供可能对构建心理模型有用的高级描述。

Raft节点始终处于以下三种状态之一：跟随者（follower），候选者（candidate）或领导者（leader）。所有节点最初都是作为跟随者开始的。在这种状态下，节点可以接受来自领导者的日志条目并进行投票。如果一段时间未收到任何条目，则节点会自动升级为候选状态。在候选状态下，节点向其对等方请求投票。如果候选人获得法定人数的选票，则将其晋升为领导人。领导者必须接受新的日志条目并复制到所有其他关注者。此外，如果过时的读取是不可接受的，那么所有查询也必须在领导者上执行。

集群拥有领导者后，就可以接受新的日志条目。客户可以请求领导添加新的日志条目（从Raft的角度来看，日志条目是不透明的二进制blob）。然后，领导者将条目写入持久性存储，并尝试复制到一定数量的关注者。一旦日志条目被认为已提交，就可以将其应用于有限状态机。有限状态机是专用的。在Consul的情况下，我们使用[MemDB](https://github.com/hashicorp/go-memdb)维护集群状态。领事的writes块直到被提交和应用为止。与查询的[一致性（consistent）](https://www.consul.io/api/features/consistency#consistent)模式一起使用时，可以实现写后读语义。

显然，不希望允许复制日志以无限制的方式增长。Raft提供了一种机制，通过该机制可以对当前状态进行快照并压缩日志。由于FSM抽象，恢复FSM的状态必须导致与重播旧日志相同的状态。这使Raft可以在某个时间点捕获FSM状态，然后删除用于达到该状态的所有日志。这是自动执行的，无需用户干预，可以防止磁盘无限制使用，同时还可以最大程度地减少重播日志所花费的时间。使用MemDB的优点之一是，即使在快照旧状态时，它也允许Consul继续接受新事务，从而避免了任何可用性问题。

共识是容错的，直到达到法定人数为止。如果没有法定数量的节点，则无法处理日志条目或有关对等成员身份的理由（reason）。例如，假设只有两个对等体：A和B。仲裁大小也为2，这意味着两个节点必须同意提交日志条目。如果A或B失败，则现在无法达到法定人数。这意味着集群无法添加或删除节点，也无法提交任何其他日志条目。这导致不可用。此时，将需要手动干预才能删除A或B并在引导模式（bootstrap mode）下重新启动其余节点。

3个节点的Raft集群可以容忍单个节点故障，而5个集群的Raft集群可以容忍2个节点故障。推荐的配置是每个数据中心运行3或5个Consul服务器。这样可以在不大大降低性能的情况下最大化可用性。本文最下方的部署表总结了潜在的集群大小选项以及每个选项的容错能力。

在性能方面，Raft可以媲美Paxos。假设领导稳定，提交日志条目需要单次往返集群的一半。因此，性能受磁盘I/O和网络延迟的约束。尽管Consul并非旨在成为高吞吐量写入系统，但它应根据网络和硬件配置每秒处理数百至数千个事务。

## Consul中的Raft

只有Consul服务器节点参与Raft，并且是对等集的一部分。所有客户端节点将请求转发到服务器。此设计的部分原因是，随着将更多成员添加到对等集合中，法定人数的数量也会增加。由于你可能正在等待数百台机器就一项输入达成共识，而不是一小撮，这会带来性能问题。

开始使用时，单个Consul服务器将进入“引导（bootstrap）”模式。该模式允许自己当选为领导。选举领导者后，可以将其他服务器添加到对等集，以保持一致性和安全性。最终，一旦添加了头几台服务器，就可以禁用引导模式。

由于所有服务器都作为对等集的一部分参与，因此它们都知道当前的领导者。当RPC请求到达非领导服务器时，该请求将转发给领导。如果RPC是查询类型，则意味着它是只读的，领导程序将根据FSM的当前状态生成结果。如果RPC是事务类型，则意味着它会修改状态，领导者将生成一个新的日志条目，并使用Raft应用它。一旦提交了日志条目并将其应用于FSM，事务就完成了。

由于Raft复制的性质，性能对网络延迟很敏感。因此，每个数据中心都选举一个独立的领导者，并维护一个不相交的对等集。数据按数据中心进行分区，因此每个领导者仅负责其数据中心中的数据。收到对远程数据中心的请求后，该请求将转发给正确的领导者。此设计可在不牺牲一致性的情况下实现较低的延迟事务和较高的可用性。

## 一致性模式

尽管所有对复制日志的写入都通过Raft，但读取更为灵活。为了支持开发人员可能需要的各种折衷，Consul支持3种不同的读取一致性模式。

三种读取模式是：

default - Raft使用领导者租赁，提供一个时间窗口，领导者在该时间窗口中扮演稳定的角色。但是，如果将领导者与其余的同伴分开，则可以在旧领导者持有租约的同时选出新领导者。这意味着有2个领导节点。由于老领导者将无法提交新日志，因此没有裂脑的风险。但是，如果旧的领导者提供任何读取服务，则这些值可能会过时。默认的一致性模式仅依赖于领导者的租赁，使客户面临潜在的过期值。我们之所以要做出这样的权衡，是因为读取速度快，通常具有很强的一致性，并且仅在难以触发的情况下才过时。由于领导者将因分区而下台，因此陈旧读取的时间窗口也受到限制。

consistent - 此模式具有强一致性，没有警告。它要求领导者与法定人数的对等节点核实它仍然是领导者。这为所有服务器节点引入了额外的往返。权衡（ trade-off）始终是一致性的读取，但是由于额外的往返行程而增加了延迟。

stale - 此模式允许任何服务器为读取提供服务，而不管其是否为领导者服务器。这意味着读取可以是过时的，但通常在领导者的50毫秒之内。权衡（trade-off ）是非常快速和可扩展的读取，但是具有过期的值。此模式允许无领导者的读取，这意味着不可用的集群仍将能够响应。

有关使用这些各种模式的更多文档，请参阅[HTTP API](https://www.consul.io/api/features/consistency)。

## 部署表

下表显示了各种集群大小的法定人数大小和容错能力（集群中可以失败的节点数）。推荐的部署是3或5台服务器。不建议使用单服务器部署，因为在故障情况下不可避免地会丢失数据。

| Servers | Quorum Size | Failure Tolerance |
| ------- | ----------- | ----------------- |
| 1       | 1           | 0                 |
| 2       | 2           | 0                 |
| 3       | 2           | 1                 |
| 4       | 3           | 1                 |
| 5       | 3           | 2                 |
| 6       | 4           | 2                 |
| 7       | 4           | 3                 |



[Consensus Protocol](https://www.consul.io/docs/architecture/consensus)

[Consul vs. ZooKeeper, doozerd, etcd](https://www.consul.io/docs/intro/vs/zookeeper)

[Consul vs. Eureka](https://www.consul.io/docs/intro/vs/eureka)