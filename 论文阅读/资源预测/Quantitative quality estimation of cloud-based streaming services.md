# Quantitative quality estimation of cloud-based streaming services

标签：计算机论文 资源预测

 Computer Communications，2018，CCF C类

目标是云上的基于流的服务，比如实时流视频和游戏服务。需要对这些服务进行实时建模，使用方法为排队网络。

## 摘要

云上的一些流服务，比如实时视频与游戏服务，已经变为了近年来比价流行的服务。在这些服务部署之前进行系统的质量估计是重要且困难的，因为服务器、客户端、网络环境的复杂与动态变化。

本文采用了排队模型来进行建模，将packet level dynamic纳入考虑，因此customer-affected performance可以用一种混合模拟的方式被估算出来。模拟方法，对于在部署服务之前估算服务质量而言是部分有用的。

分析模型可以分为两个部分：
1. VM级别的服务排队模型，带有对平均顾客数量、平均等待时间、平均部署VM数量的静态的闭环表示。
2. 客捕获流数据包滞后时间的客户端处的微观模型和仿真过程

模拟过程基于分析模型产生，模拟的结果显示服务质量是如何被服务器和用户性能影响的，提供了一种不同的视角来看待云资源的提供与用户参数设置。

## Introduction

流服务是基于session的，可能会有以下的问题：
1. 服务中断（虚拟机经常在处理用户请求时临时停机）
2. 长时间的延迟（需要较长时间等待虚拟机去恢复功能）

可以得到的信息（prior assessment）：
* arrival rate of demand
* distribution of the length of connection sessions

云服务提供商需要预测：
1. 需要的虚拟机数量，对于一个给定的操作集合而言
2. 对服务级别的估计，比如服务中断以及客户端延迟的平均数据

考虑到一个简单的操作集合：
* 所有系统中的资源被认为是活跃的
* 每一个到来的客户会被分配给流量最小的VM（或是在流量未知时分配给用户数最少的VM）

此时，将VM建模为服务器，客户的到达速率未知，产生的服务未知，因此服务的处理时间也是未知的。即使参考排队论中的经典假设，比如泊松到达流和指数服务时间（连接时间），此时所有的随机变量都是相互独立的，这种问题是没有closed-form solution的。因为数量过大，统计学上的估计是难以进行的。

作者认为，在这里closed-form solution的缺乏和精度的缺失其实并不是主要的问题。问题的关键是传统的排队论模型将VM建模为服务器来为顾客服务并没有抓住问题的本质。它没有对基于云的服务系统的物理层的动态变化进行建模，并忽视了这种动态变化与系统的性能在服务层级上的联系。因此在下一段我们首先会描述基于云的服务系统的物理层，并解释传统排队模型中存在的问题。

对于流服务来说，它有很多的特殊性……（太多不想翻译了）。我个人觉得最重要的地方在于，服务器在轮到某个用户时建立的虚拟链接是动态变化的，这使得包的发送并不是按照顺序进行的。对于流服务的客户端来说，如果较早的包没有到达，是无法跳过的，会产生延迟。

在将VM作为模型、包作为顾客的传统排队论建模中，它完全没有考虑到基于云的系统的物理层。本文我们使用了一种分析的方法来显示地考虑基于云的系统中的物理层的变化。

## 假设

1. 用户请求的到来服从静态的速率为$\lambda$的泊松过程
2. 用户连接到云服务器的连接时间是与请求过程独立的
3. VM采用round-robin的方式来服务客户，每一次服务时间为一个固定的时间间隔quantum，用$\Delta$表示
4. VM被编程按照以下步骤来初始化和退役：任何时刻只会有一台VM接受新的请求。每一步中对于接受数量少于B的VM会进行退役，首先是不再允许新的用户加入并将其纳入退役列表，在所有用户退出后放入空置VM池中；其次，在有新的请求到来时激活一台VM并要求分发器向其发送更多请求。

泊松到达过程是对大量用户请求的近似建模，即到达时间迥异，但是平均到达速率基本不变。但现实中的情况一般是以天或周为单位的周期性请求变化，因此对本文这个假设的一个解释是Green and Kolesar的simple peak hour approximation，即用每个小时的最高请求率作为一个静态的泊松过程的参数。这样可以作为不在顶点时最差情况的一个估计。

PS：steady-state approximation for time-dependent performance这个问题的讨论有很多，见原文引用

第二个假设在很多经典的排队论模型中都有见到。

## 基于云的流量系统中的动态变化

基于云的系统的分析是非常困难的，对于一个给定需求模式所需要的VM数量的分析，只能通过利用统计学上不精确的结果来模拟大型的、复杂的系统，从而不断增进认识来推导出来。

上述方法只能够推出一个平均值，比如对于独立客户在单个VM上的经验的模拟。这使得云服务提供者在使用资源时有很大的空间。

根据四个假设，虚拟机的状态有三种：
* working(初始化后)
* retiring(在用户的数量超过上阈值B后)
* idling（在前一阶段的B个用户全部完成session后）

在任何阶段都只有一个对应的worker来服务到来的用户，然而在任何时候可以同时有多个worker在工作。retiring VM实际上是在工作的。

用户的到来服从一个泊松过程，从请求到达后到服务器开始处理前会经历一个短的时间。所有的VM会采用round-robin方式来进行负载均衡。而每一个用户实际上是分得了一个时间片来进行服务的。所有如果有一个用户能够长期被一个VM服务，只可能是这个VM的队列中就只有这么一个用户。

用$\phi$指代客户的一般连接时间(generic connection time of a customer)，即用户愿意在基于云的服务中花费的时间。这个时间包含了lag time（等待服务的时间）。根据假设，这个连接时间与到达速率$\mu$是独立同分布的。（指数分布）

显然，系统的操作特性和用户的服务级别是由两个控制参数决定的，一个是VM同时能服务的最大客户数量B，另一个是VM给每个任务分配的单位处理时间$\Delta$。系统中任务的到达速率$\lambda$和服务处理速率$\mu$同样会对系统的特性产生影响。

在本文中，使用了一种混合的模拟方法来估算顾客在给定参数下的服务级别。这种方式是分为两个级别的，宏观级别考虑VM和顾客之间的交互，微观级别考虑用户和基于云的服务的物理层直接的交互。

在分析系统的是，我们只会关注于与VM本身，而忽视dispatcher和monitor。系统会表现地像是外部的请求会直接到达VM而不经过负载均衡器。同样，monitor会得到足够多的资源。

3.1会建立一个马尔科夫模型；3.2给出对应worker的一个静态分布用来推导出操作的特性和系统的服务级别；3.3和3.4分别给出两种系统的操作特性，VM的平均生存时间和系统所需要的平均VM数量。

### 3.1 指数连接时间的转移

假设$s=(n;n_1,...,n_B)$作为系统的状态，n为对应worker所处理的顾客数量，n是一个比B小的整数。$n_j$是由j个顾客的retiring机器的数量，j是1~B的一个整数。

* $s_+$为$(n+1;n_1,...,n_B)$，表示一个多一个顾客的状态，
* $s_-$表示少一个顾客的状态
* $s_R=(0;n_1,...,n_B)$表示没有顾客且客户数为B的实例数比s多1
* $s_{j-}=(n;n_1,...,n_{j-2},n_{j-1}+1,n_{j}-1,...,n_B)$表示有一个顾客数为j的retiring VM的客户降低了。
* $s_{1-}=(n;n_1-1,...,n_B)$表示有一个1顾客的机器中顾客减少退出了系统。

注意由假设有，连接时间与顾客的到来是独立的。如果两个顾客在某一时刻位于同一个VM中，则顾客从VM离开的瞬时速率是$2\mu$(the instantaneous rate of the departure of customers from the VM is of 2\mu)，而不是标准共享处理器系统中的$\mu$，因此对状态s进行估算：

* Total rate out: $q_s=[\lambda+(n+\sum^B_{j=1}j\times n_j)\times \mu]$
* Effect of an arrival，对于到来的影响
    * $q_{s,s_+}=\lambda$，对于$n\leq B-2$
    * $q_{s,s_R}=\lambda$，对于$n= B-1$
* Effect of a departure，对于离开的影响
    * $q_{s,s_-}=n\times \mu$
    * $q_{s,s_{j-}}=n_j\times\mu\times j$，对于$n_j\ge 1$，j属于1~B

但是q是什么？

### 3.2 当前worker状态的静态分布

连续马尔科夫链的图略，自行查阅论文Fig1

令$p_i$代表状态i的概率，i取0~B-1。有以下等式：
* $\lambda\times p_B+\mu \times p_0 = \lambda \times p_0$，即B状态的到达速率与1状态的离开速率应该等于0状态的到达速率（B状态后VM retiring）
* $\lambda\times p_{i-1}+(i+1)\times \mu \times p_{i+1}=(\lambda + i\times \mu)\times p_i,\forall i \in(1,...,B-2)$，这是从i状态的出入平衡考虑的
* $\lambda\times p_{B-2}=(\lambda +\mu\times(B-1))\times p_{B-1}$，这是从B-1状态的出入平衡考虑（B-1状态只能从B-2状态到达）

令$\xi=\lambda/\mu$作为到达率和离开率的比例，$\xi$则指代了系统在M/M/1队列中的繁忙程度。当$\xi < 1$时称系统是空闲的，反之则是繁忙的。通过求解上述三条等式，有

$p_{B-k}=[\sum_{m=1}^k(\prod_{j=m}^{k-1}(\frac{B-j}{\xi^{k-m}}))]\times p_{B-1}$

公式不写了，看不懂也没意思。最终计算出平均顾客数量$L=\sum_{k-1}^{B-1}p_k \times k$，不过这个是用M/M/1推出来的。

显然，$\xi$和B能够决定L的值。然后用这两个值进行了一系列的模拟，最终发现在系统繁忙的时候，随着服务速率的增加，L先增加后减少，存在一个拐点。

观察到下面三种情况

* L像是$\xi$在系统繁忙时的一个单峰函数（mono-peak function）
* B增长时，L达到最大值时$\xi$的值也在增长。
* 在空闲系统(lazy system)中，L与$\xi$比较接近。