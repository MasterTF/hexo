---
title: Quorum机制
date: 2021-01-16 16:51:03
tags:
    - 分布式系统
    - quorum
categories:
    - 技术
    - 分布式系统
typora-root-url: ../../source
typora-copy-images-to: ../../source/img
---
## WARO机制介绍
先看一个极端的情况：WARO(Write All Read one)是一种简单的副本控制协议，当Client请求向某副本写数据时(更新数据)，只有当所有的副本都更新成功之后，这次写操作才算成功，否则视为失败。

<!--more-->

从这里可以看出两点：
1. 写操作很脆弱，因为只要有一个副本更新失败，此次写操作就视为失败了。
2. 读操作很简单，因为，所有的副本更新成功，才视为更新成功，从而保证所有的副本一致。这样，只需要读任何一个副本上的数据即可。
假设有N个副本，N-1个都宕机了，剩下的那个副本仍能提供读服务；但是只要有一个副本宕机了，写服务就不会成功。

从上述分析可以发现 WARO 读服务的可用性较高，但更新服务的可用性不高，甚至虽然使用副本，但更新服务的可用性等效于没有副本。

## Quorum机制介绍
- Quorum是分布式系统中用来保证数据冗余和最终一致性的投票算法，其主要数学思想来源于鸽巢原理。
- 下面将 WARO 的条件进行松弛，从而使得可以在读写服务可用性之间做折中，得出 Quorum 机制。
- 假设有N个副本，更新操作wi 在W个副本中更新成功之后，才认为此次更新操作wi 成功。称成功提交的更新操作对应的数据为：“成功提交的数据”。对于读操作而言，至少需要读R个副本才能读到此次更新的数据。其中，W+R>N ，即W和R有重叠。一般，W+R=N+1
 ![](/img/16211552541601.jpg)

举例：
1. 假设系统中有5个副本，W=3，R=3。初始时数据为(V1，V1，V1，V1，V1）--成功提交的版本号为1
2. 当某次更新操作在3个副本上成功后，就认为此次更新操作成功。数据变成：(V2，V2，V2，V1，V1）--成功提交后，版本号变成2
3. 因此，最多只需要读3个副本，一定能够读到V2(此次更新成功的数据)。而在后台，可对剩余的V1 同步到V2，而不需要让Client知道。

Quorum 机制的三个系统参数 N、W、R 控制了系统的可用性，也是系统对用户的服务承诺:数 据最多有 N 个副本，但数据更新成功 W 个副本即返回用户成功。对于一致性要求较高的 Quorum 系 统，系统还应该承诺任何时候不读取未成功提交的数据，即读取到的数据都是曾经在 W 个副本上成 功的数据。

## Quorum机制分析
### 仅依赖Quorum机制无法保证强一致性
- 所谓强一致性就是：任何时刻任何用户或节点都可以读到最近一次成功提交的副本数据。强一致性是程度最高的一致性要求，也是实践中最难以实现的一致性。
- 因为，仅仅通过Quorum机制无法确定最新已经成功提交的版本号(除非将最新已提交的版本号作为元数据由特定的元数据服务器或元数 据集群管理)。
- 比如，上面的V2 成功提交后（已经写入W=3份），尽管读取3个副本时一定能读到V2，如果刚好读到的是(V2，V2，V2），则此次读取的数据是最新成功提交的数据，因为W=3，而此时刚好读到了3份V2。
- 如果读到的是（V2，V1，V1），则无法确定是一个成功提交的版本，还需要继续再读，直到读到V2的达到3份为止，这时才能确定V2 就是已经成功提交的最新的数据。
![](/img/16211553483856.jpg)

### 如何通过quorum达到强一致性
1. 限制提交的更新操作必须严格递增，即只有在前一个更新操作成功提交后才可以提交后一 个更新操作，保证成功提交的数据版本号必须是连续增加的
2. 如何读取最新的数据？---最多读R个副本就可以读到最新的数据了。
3. 如何确定最高版本号的数据是一个成功提交的数据？---继续读其他的副本，直到读到的 最高版本号副本 出现了W次。
4. 若不满足，则R中版本号第二大的为最新成功提交的副本

实际工程中，应该尽量通过其他技术手段，回避通过 Quorum 机制读取最新的成功提交的版本。例如，当 quorum 机制与 primary-secondary 控制协议结合使用时，可以通过读取 primary 的方式读取到 最新的已提交的数据。

### 基于 Quorum 机制选择 primary
选择新的 primary 的工作是由某一额外的中心节点完成。

中心节点读取 R 个副本，选择 R 个副本中版本号最高的副本作为新的 primary。新 primary 与至少 W 个副本完成数据同步后作为新 的 primary 提供读写服务。
- R 个副本中一定蕴含了最新的成功提交的数据（即要么是V2,要么是V1）
- 虽然不能确定最高版本号的数是一个成功提交的数据，但新的 primary 在随后与 secondary 同步数据，使得该版本的副本个数达到 W，从而使得该版本的数据成为成功提交的数据

举例：
- 在 N=5，W=3，R=3 的系统中，某时刻副本最大版本号为(v2 v2 v1 v1 v1)，此时 v1 是系统的最新的成功提交的数据，v2 是一个处于中间状态的未成功提交的数据。
- 假设此刻原 primary 副本异常，中心节点进行 primary 切换工作。这类“中间态”数据究竟作为“脏数据”被删除，还是作为新的数据被同步后成为生效的数据，完全取决于这个数据能否参与新 primary 的选举，分两种情况：
![](/img/16211554996958.jpg)

1. 若中心节点读取到的版本号为(v1 v1 v1)，则任 选一个副本作为 primary，新 primary 以 v1 作为最新的成功提交的版本并与其他副本同步，当与第 1、 第 2 个副本同步数据时，由于第 1、第 2 个副本版本号大于 primary，属于脏数据。
![](/img/16211555196393.jpg)
2. 若中心节点读取到的版本号为(v2 v1 v1)，则选取版本号为 v2 的副本作为新的 primary。之后，一旦新 primary 与其他 2 个副本完成数据同步，则符合 v2 的副本个数达到 W 个，成为最新的成功提交的副本，新 primary 可以提供正常的读写服务。



