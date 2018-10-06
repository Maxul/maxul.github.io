**题目： ShieldBox: Secure Middleboxes using Shielded Execution**

**作者： Bohdan Trach, Alfred Krohmer, Franz Gregor, Sergei Arnautov, Pramod Bhatotia, Christof Fetzer**

**单位： Technische Universitat Dresden and University of Edinburgh**

**出版： SOSR 2018**

# 问题：

1.    Middlebox掌管网络流量，要求高效、可靠和安全。现在的网络基础架构托管在易于管理、可弹性伸缩、高容错率的云上。其中NFV已经是边缘计算的核心要素。
2.    矛盾在于：处理机密数据的middlebox运行在不可信的云上。如何在不可信的第三方平台上对网络中间件进行保护，而不牺牲太大性能，同时支持通用商业化NF呢？


# 特点：（本文是欧洲团队各种工作的大集成）

*    安全：SGX保护机密性和完整性；
*    高效：SCONE + Intel DPDK，轻量级架构，在用户态enclave下处理包；
*    通用：CLICK作为NF通用框架，允许从NIC直接取包；
*    透明：方便移植与部署。


# 威胁模型：

1.    一个强大的对手，控制计算和存储单元，试图窃取加密流和配置信息。
2.    可嗅探任意未保护内存，无法阻止拒绝服务攻击。


# 设计优点：

1.    共享内存，允许多个Enclave从NFV安全取包；
2.    用NIC PTP时钟作为可信时钟源；


# 架构：
原文图1。
之所以DPDK不能完全载入EPC，因为DPDK为了加速使用hugepage，而SGX EPC不支持。

1.     由于SGX不支持内存共享，也不支持NUMA架构（注：这些都是DPDK的优势，本文尽量想保持其优点），因此借鉴了DPDK自带的ring，在未保护的巨页上实现无锁的FIFO队列，作为并发进程的高效共享渠道。同理（和我们的设计思路一样），需要对各进程独立的数据结构进行所有权（ownership）标记。
2.     MiddleBox的哪些状态需要保护？计数信息、以太网交换机端口映射、活动日志、路由表、分析日志等等，利用SCONE文件系统模块，将数据seal后放入外存储中。
3.     以太网网卡IC日益强大，其自带时钟源解决了Enclave无可信时钟的问题。网络作为分布式系统同步的时钟源（IEEE 1588PTP协议），该思路直接而自然。
4.     Iago attacks：该攻击发生在不可信代码与可信代码的交互上，不可信者借助输入破坏（subvert）可信逻辑，常规手段是在可信部分对所有输入都做合法化检查。针对DPDK使用的巨页场景，需要对指针索引的范围做检查，保证数据只拷贝入Enclave的地址空间。一旦发现违例，即认为（assume）攻击存在，丢包并通知应用。


# 实现：

*    系统组件：DPDK+CLICK+SCONE(提供了mmap语义)
*    运行时：musl-libc+Boost C++
*    加密：WolfSSL


# 关于TCB：
* 作者没有提其代码量，但说明其静态编译为6MB。考虑x86架构下的Linux内核，6~7MB左右，个人猜测，整体代码量十分巨大。

# 优化：

-    内存预分配
-    GCC Branch Hints：Likely / Unlikely
-    用比较和减法代替取模（module）运算
-    队列优化：用DPDK  rte_ring替换C++ std:vector
-    时钟事件调度器优化


# 性能评估：

-    吞吐：256字节胜过原生（因为对DPDK做了调度优化）。但内存拷贝时间和分配释放时间占大头，连累吞吐。
-    延迟：没有系统调用RTT开销，总体引入约1.8微秒。
-    扩展性：受约束，DPDK在超线程下表现不好，多线程下对Enclave线程的抢占开销也比较可观（注：缺乏对多线程模型的优化）。
-    以多端口IP路由和IDS为case study。


# 局限：

1.    只支持OSI七层模型的L2到L3，L4到L6不支持。（我们的协议栈做了从NIC到APP的无缝链接）。
2.    文章提到，共享内存模型面临威胁，因此需要读取头部后立即处理掉。对于Time Window可能存在的中间人攻击，缺乏考虑。



# 体会：

*    迄今为止与我们最接近的工作。虽然目的（Secure IO and SGX Apps）和方法（SGX + SMM）不一致，但在设计和安全考虑、性能考虑上有多处重合。
*    我们的工作不受其启发，有自己的思考领域和专项。
*    本文工作考虑还是相当到位的，论述细致，致敬！



# 思考：

*    如果Enclave可以和可信外设建立安全信道，那么如何利用硬件为Enclave提供可信的时钟源？考虑是否可以将RTC路由给Enclave，或者直接在BIOS中为Enclave构筑时间戳。
