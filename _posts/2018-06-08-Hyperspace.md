**题目： Racing in Hyperspace: Closing Hyper Threading Side Channels on SGX**

**作者： Guoxing Chen, Wenhao Wang, Tianyu Chen, Sanchuan Chen, Yinqian Zhang, XiaoFeng Wang, Ten-Hwang Lai, Dongdai Lin**

**单位： The Ohio State University, Indiana University Bloomington & SKLOIS, Chinese Academy of Sciences**

**出版： IEEE Symposium on Security and Privacy 2018**

# 背景：
* Intel处理器上元器件众多，多种特性提供了体系结构的加速支持。该情况下，SGX技术的安全性很难保证。Spectre和meltdown即为明显例子。
* 可惜的是，超线程hyper-threading允许一个物理核同时执行两个线程代码，这种流水线复用技术也带来了侧信道。
* 分析人员总结发现，在AEX发生时，体系结构有如下组件需要关注：

1.    Store Buffer：线程私有，不存在泄露可能；
2.    FPU浮点计算单元：虽然共享，但不是所有程序都使用；
3.    TLB快表：共享，但AEX发生是CPU自动将其清空；
4.    Caches：共享，AEX发生是不会自动清空，存在LLC攻击可能；
5.    BPU分支预测单元：共享，AEX发生时也不会清空，泄露控制流信息。

已有工作通过TSX事务内存技术来保证别的物理核无法访问enclave核的物理硬件单元数据，然而，超线程技术的多个线程却不受TSX控制。
也就是说，Intel上的HT双线程中，一个非enclave线程（攻击者）可以窃取enclave线程（受害者）的信息！

# 解决问题：
既然HT会给SGX的安全性带来危害，一个直接想法就是将其关闭。

那么我们怎么知道目标机器是否开启了HT呢？

1.    由于没有办法在enclave中调用cpuid、rdtscp和rdpid等特殊指令，我们无法知道cpu提供了什么支持，开启了什么特性；
2.    另外，远程验证也不包含HT的任何信息，IAS无从判断；
3.    另一个笨方法是使用shadow线程和enclave共享一个物理核，抢占其他攻击者的位置，但是谁能保证调度器诚实地做到physical-core co-location呢？

本文就是寻找笨方法下如何验证被保护线程PT和影子线程ST同时运行在物理核上的工作。

# 方案设计：

1.    和检测AEX发生机制的Deja Vu工作一样，本文基于概率算法。共享物理核心的硬件线程本质共享了L1 Cache，因此时间窗口可以设计得足够小。PT和ST共享一全局可见变量，轮流修改，之间设计的busy-wait设计在10个周期左右，就能确定PT和ST是否共用核心。一旦分别运行在两个核上，由于硬件Cache Coherence Protocol存在的原因，访存速度变慢，延迟大概在190个周期左右，就会有一个线程发现10个周期过后数据没有被对方修改，由此检查攻击！
2.    接着基本是数学运算了：提出基于假设（hypothesis）的模型，将实验运行得到的empirical经验值和theoretical理论值比较，利用公式做判决即可。（这些团队成员具备较强数学素养，应该有通信背景，模型基本来自于统计学和随机过程等。）


# 攻击测试：
攻击者修改CPU周期，禁止Cache等手段修改机器行为。看检测模型是否依旧有效。

基于四个测试：

    1. 单个核上cache eviction发生时的计算延迟，
    2. 跨核心通信时的计算延迟，
    3. CPU运行速率修改后模型的正确率，
    4. 关于cache机制后模型的正确性。


# 实现方式：

1.    LLVM插桩。
2.    得到控制流图，在原运行的enclave指令中插入变量修改代码，同时生成一个新线程作为影子线程，最后链接HyperRace检测库。


# 性能测试：
* 开销来源：

    1. 运行时：多了一个线程，多个变量读写和检测机制；
    2. 内存占用：插桩导致code base增加，内存使用增加；

利用nbench做测试，最大在37.7%。

# 结论：

*    Performance oriented hardware design leads to security implications.



# 体会：

1.    本文在笨方法上做突破，虽然想法简单，但是得到了认可。“黑猫白猫”理论可以借鉴。
2.    检测方法的结论是：检测效果越好，引入开销越大。这个方式不合理。
3.    本文很多工作是在AsiaCCS  DEJA VU上完成，前者做了AEX detection，本文做了hyper thread detection (co-location test)，方法一样，思路一样。由AsisCCS发展到S&P，非常值得学习。
