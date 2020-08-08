---
title: 2PC And 3PC
date: 2019-11-03 16:40:28
categories:
    - Internet
tags:
    - 2PC
    - 3PC
    - transaction
description: "Distributed System 分布式系统中数据、事务、指令、操作的同步一致性至关重要。本文介绍了2PC与3PC的概念与原理"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


## 2PC——二阶段提交
> 分布式数据要尽可能保证一致性，2PC即二阶段提交就是保证数据一致性的一种手段。

在分布式系统中，每个节点虽然可以知晓自己的操作时成功或者失败，却无法知道其他节点的操作的成功或失败。
当一个事务跨越多个节点时，为了保持事务的ACID特性，需要引入一个作为协调者的组件来统一掌控所有参与者的操作结果并最终指示这些节点是否要把操作结果进行真正的提交。
因此，二阶段提交的算法思路可以概括为：参与者将操作成败通知协调者，再由协调者根据所有参与者的反馈情报决定各参与者是否要提交操作还是中止操作。

所谓的两个阶段是指：第一阶段：准备阶段和第二阶段：提交阶段

### 准备阶段
事务协调者给每个参与者发送Prepare消息，每个参与者要么直接返回失败，要么在本地执行事务，但不提交。
可以进一步将准备阶段分为以下三个步骤：
* 1）协调者节点向所有参与者节点询问是否可以执行提交操作，并开始等待各参与者节点的响应。
* 2）参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志。
* 3）各参与者节点响应协调者节点发起的询问。如果参与者节点的事务操作实际执行成功，则它返回一个”同意”消息；如果参与者节点的事务操作实际执行失败，则它返回一个”中止”消息。

![](2PC与3PC_1.png)

### 提交阶段
如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚消息；否则，发送提交消息；参与者根据协调者的指令执行提交或者回滚操作。
接下来分两种情况分别讨论提交阶段的过程。
* 1）当协调者节点从所有参与者节点获得的相应消息都为”同意”时:

![](2PC与3PC_2.png)
1. 协调者节点向所有参与者节点发出”正式提交”的请求。
2. 参与者节点正式完成操作，并释放在整个事务期间内占用的资源。
3. 参与者节点向协调者节点发送”完成”消息
4. 协调者节点受到所有参与者节点反馈的”完成”消息后，完成事务。

* 2）如果任一参与者节点在第一阶段返回的响应消息为”中止”，或者 协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时：

![](2PC与3PC_3.png)
1. 协调者节点向所有参与者节点发出”回滚操作”的请求。
2. 参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源。
3. 参与者节点向协调者节点发送”回滚完成”消息。
4. 协调者节点受到所有参与者节点反馈的”回滚完成”消息后，取消事务。

### 2PC 的缺点
1. 同步阻塞问题。执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。（当参与者收到组织者的消息之后，需要登录游戏，在游戏中等待组织者的再次邀请，这个过程比较浪费时间。）
2. 单点故障。由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）
3. 数据不一致。在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据部一致性的现象。
4. 二阶段无法解决的问题：协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

为了解决以上缺点就有了3PC

## 3PC——三阶段提交
> 所谓3PC，就是把2PC的准备阶段再次一分为二，组成了三阶段。
3PC并没彻底解决2PC存在的所有问题
<details>
<summary> 2PC举例 </summary>
组织者：小A，我们准备玩王者荣耀，你要是可以来参加的话，现在你就登录游戏，然后在游戏好友上给我回复个消息。

小A登录自己的游戏账号，然后告诉组织者：小A已就位。

组织者：小B、小C、小D，我们准备玩王者荣耀，你要是可以来参加的话，现在你就登录游戏，然后在游戏好友上给我回复个消息。

小B、小C、小D分别登录自己的游戏账号，然后告诉组织者：小B、小C、小D已就位。

组织者发现所有人都就位了，于是在游戏上逐一通知大家，

组织者：小A，我邀请你了，你进来吧。

小A接受邀请

组织者：小B、小C、小D，我邀请你了，你进来吧。

小小B、小C、小D接收邀请
</details>

<details>
<summary> 3PC举例 </summary>
组织者：小A，我们想定在晚上8点，你有时间嘛？有时间你就说YES，没有你就说NO，然后我还会再去问其他人，这段时间你可先去干你自己的事儿，不用一直等着我。

小A：好的，我有时间。

组织者：小B、小C、小D，我们想定在晚上8点王者荣耀五黑……不用一直等我。

组织者收集完大家的时间情况了，一看大家都有时间，那么就再次通知大家。（协调者接收到所有YES指令）

组织者：小A，我们确定了晚上8点王者荣耀五黑，你要把段时间空出来，你不能再安排其他的事儿了。然后我会逐个通知其他朋友，通知完之后我会再来和你确认一下，还有啊，如果我没有特意给你打电话，你就8点上号就行了。对了，你确定能来是吧？

小A顺手设置了晚上8点闹钟，然后跟组织者说，我可以去。

组织者：小B，我们决定了晚上8点王者荣耀五黑……你就8点上号就行了。

组织者通知完一圈之后。所有朋友都跟他说：”我已经把8点这个时间段空出来了”。于是，他在8点的时候这一天又挨个打了一遍电话告诉他们：嘿，现在你们可以上号啦。。。。

小A、小B、小C、小D：我已经登录了，你拉我吧。

组织者邀请A、B、C等加入游戏。
</details>

和2PC相比，3PC多了一个步骤，就是提前询问所以参与者是否都能参与，并且所有人都同意后再次通知大家登录游戏。

在第一阶段，只是询问所有参与者是否可以执行事务操作，并不在本阶段执行事务操作。当协调者收到所有的参与者都返回YES时，在第二阶段才执行事务操作，然后在第三阶段在执行commit或者rollback。

这样三阶段提交就有CanCommit（事务询问）、PreCommit（事务执行）、DoCommit（事务提交）三个阶段。

### CanCommit阶段
3PC的CanCommit阶段其实和2PC的准备阶段很像。协调者向参与者发送commit请求，参与者如果可以提交就返回Yes响应，否则返回No响应。

![](2PC与3PC_4.png)
1、事务询问：协调者向参与者发送CanCommit请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。

2、响应反馈：参与者接到CanCommit请求之后，正常情况下，如果其自身认为可以顺利执行事务，则返回YES响应，并进入预备状态。否则反馈NO

### PreCommit阶段
协调者根据CanCommit阶段参与者的反应情况来决定是否可以进行事务的PreCommit操作。
假如协调者从所有的参与者获得的反馈都是YES响应，那么就会执行事务的预执行：

![](2PC与3PC_5.png)
1、发送预提交请求：协调者向参与者发送PreCommit请求，并进入Prepared阶段。

2、事务预提交：参与者接收到PreCommit请求后，会执行事务操作，并将undo和redo信息记录到事务日志中。

3、响应反馈：如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。

假如有任何一个参与者向协调者发送了NO响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。

![](2PC与3PC_6.png)
1、发送中断请求：协调者向所有参与者发送abort请求。

2、中断事务：参与者收到来自协调者的abort请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。

### doCommit阶段
该阶段进行真正的事务提交，也可以分为以下两种情况。
如果协调证收到所有参与者的事务执行后的ACK响应，则发生如下事情：

![](2PC与3PC_7.png)
1、发送提交请求：协调接收到参与者发送的ACK响应，那么他将从预提交状态进入到提交状态。并向所有参与者发送doCommit请求。

2、事务提交：参与者接收到doCommit请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。

3、响应反馈：事务提交完之后，向协调者发送Ack响应。

4、完成事务：协调者接收到所有参与者的ack响应之后，完成事务。

如果协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务。

![](2PC与3PC_8.png)
1、发送中断请求：协调者向所有参与者发送abort请求

2、事务回滚：参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。

3、反馈结果：参与者完成事务回滚之后，向协调者发送ACK消息

4、中断事务：协调者接收到参与者反馈的ACK消息之后，执行事务的中断。

还有一种情况，如果参与者无法及时接收到来自协调者的doCommit或者abort请求时，会在等待超时之后，会继续进行事务的提交。

![](2PC与3PC_9.png)

## 3PC比2PC好在哪？
### 降低同步阻塞
在3PC中，第一阶段并没有让参与者直接执行事务，而是在第二阶段才会让参与者进行事务的执行。大大降低了阻塞的概率和时长。并且，在3PC中，如果参与者未收到协调者的消息，那么他会在等待一段时间后自动执行事务的commit，而不是一直阻塞。

### 提升了数据一致性

2PC中有一种情况会导致数据不一致，如在2PC的阶段二中，当协调者向参与者发送commit请求之后，发生了网络异常，只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据不一致性的现象。

这种情况在3PC的场景中得到了很好的解决，因为在3PC中，如果参与者没有收到协调者的消息时，他不会一直阻塞，过一段时间之后，他会自动执行事务。这就解决了那种协调者发出commit之后。

另外，2PC还有个问题无法解决。那就是协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

这种情况在3PC中是有办法解决的，因为在3PC中，选出新的协调者之后，他可以咨询所有参与者的状态，如果有某一个处于commit状态或者prepare-commit状态，那么他就可以通知所有参与者执行commit，否则就通知大家rollback。因为3PC的第三阶段一旦有机器执行了commit，那必然第一阶段大家都是同意commit的，所以可以放心执行commit。
## 3PC无法解决的问题
在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者abort请求时，会在等待超时之后，会继续进行事务的提交。

所以，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

所以，我们可以认为，无论是二阶段提交还是三阶段提交都无法彻底解决分布式的一致性问题。

Google Chubby的作者Mike Burrows说过：

there is only one consensus protocol, and that’s Paxos” – all other approaches are just broken versions of Paxos。

意即世上只有一种一致性算法，那就是Paxos，所有其他一致性算法都是Paxos算法的不完整版。