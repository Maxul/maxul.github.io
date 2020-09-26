## SGX论文回顾

### Using Innovative Instructions to Create Trustworthy Software Solutions | HASP@ISCA'13

这是介绍SGX指令的第一篇文章。
大致将SGX ISA分为用户态（ENCLU）和内核态（ENCLS）两种，用户态负责模式切换、密钥和认证管理，内核负责全生命周期和换页等机制。
头一次在x86商业硬件中宣示加入反向沙盒的能力对云计算领域是不小的革新。

### Shielding Applications from an Untrusted Cloud with Haven | OSDI'14

微软研究院率先拿到SGX芯片，并尝试借助Library OS的思路将应用保护在SGX中。
设计中最大的亮点是在App和底层不可信OS之间加入一层检查层（shim），对系统调用的参数和返回值进行检查。
同时借助LibOS减少对底层OS的依赖，从而降低对外暴露的攻击面。
文章还对SGX2代的指令集进行了讨论，以及当前硬件能力局限导致的兼容性、可用性、安全性不足进行了讨论，启发了不少后续工作。

### VC3: trustworthy data analytics in the cloud using SGX | S&P'15

依旧是微软研究院的工作。将SGX用在了分布式领域，保护MapReduce大数据计算模型。
VC3设计了针对SGX访存模型的编译器，保证数据始终不写出可信边界。
文章还讨论了如何验证数据流计算正确性的问题，和之后的Ryoan十分相似。

### Moat: Verifying Confidentiality of Enclave Programs | CCS'15

将形式化验证的方法对准了SGX程序，证明该程序不会显式将隐私泄露出去。

### Controlled-Channel Attacks: Deterministic Side Channels for Untrusted Operating Systems | IEEE S&P 2015

微软研究院的力作。因为SGX的页表仍由内核管理，内核可以精准地基于缺页中断（page fault）监视当前SGX实例（即enclave）的控制流和数据流轨迹。
对于确定的输入，SGX实例会有确定的行为和输出，由此可以窥知SGX中的秘密。
文章借助对开源软件提供的代码透明性，可以提前知道SGX实例的内存访存行为，实验达到了68%到90%的的秘密还原能力，基本摧毁了人们对SGX安全的假设。安全人员也由此开启了对SGX的进攻之路。
文章探索了几种可能的防御措施，基本在后续都有follow-up工作出现，是名副其实的“种子文章”。

### SCONE: Secure Linux Containers with Intel SGX | OSDI'16

对SGX的易用性做了支持，将容器技术和SGX进行了结合。
其实现思路和Library OS相似，分别对文件系统、网络和终端设计了3个shim层。
在性能优化方面，借助用户态线程（类似协程）和FlexSC的减少模式切换。

### Ryoan: a distributed sandbox for untrusted computation on secret data | OSDI'16

顾名思义，分布式沙盒。其场景选取得很好，即用户的数据需要借助第三方的计算得出结果，如基因测序、人脸识别或邮件过滤等。
数据拥有者和算法提供方选择在公有云上完成这笔计算，这里用户希望保证计算过程中数据不能泄露，当然算法也可以在保护范畴之内。
Ryoan借助浏览器中使用NaCl沙盒技术保证数据不离开安全内存边界，对于分布式数据流使用有向无环图（DAG）来保证最终结果的正确。
在性能加速层面，Ryoan将自己标注为面向请求的服务模型，类似于微服务，借助Checkpoint/Restore来优化服务的速度。
Ryoan使用的SGX2代的指令集（基于模拟器），即动态内存管理。

### SGX Support for Dynamic Memory Management Inside an Enclave | HASP'16

Intel针对1代的问题，对SGX进行了扩展。主要解决的是运行时内存地址空间无法扩大的弊病。
SGX2 ISA依旧让内核来完成EPC的管理工作，但是用户态可以对内核的内存管理行为进行检验，即trust but verify。

### AsyncShock: Exploiting Synchronisation Bugs in Intel SGX Enclaves | ESORICS'16

SGX的系统服务基本依赖于内核，进程调度也不例外。
文章讨论了利用并发漏洞窃取SGX秘密的可能。借助用户态调度可以有效避免这一点。

### Eleos: ExitLess OS Services for SGX Enclaves | EuroSys'17

这是篇来自以色列理工的性能加速工作。基本思路和OSDI'10的FlexSC非常相似，该方案在SCONE中也有提及。
由内核来完成EPC的换页工作不仅有安全问题（Controlled-Channel Attacks），而且还有性能问题：一旦发生EPC的page fault，就必须切出SGX模式，进入内核进行换页。EPC频繁换页的根本原因在于SGX物理内存资源极为有限（128MiB或256MiB）。
观察到SGX内的代码可以直接访问外部不受保护的DRAM，那么将EPC的换页工作从硬件改为软件可以消除模式切换引入的开销。
Eleos将之前应用于GPU上的软件页表翻译工作Active Pointer (ISCA'16)用在了SGX里，让memcached在同等开销下能承载了更多加密数据。

### SGXBounds: Memory Safety for Shielded Execution | EuroSys'17

SGX内的程序同样会有内存安全的问题。SGXBounds借助Tagged Pointers对SGX程序进行访存保护。
这里的观察在于SGX的1代实现只提供了36位的地址空间，即64GiB，因此64位寻址下高位地址可以用来检查指针的访存范围。

### Hybrids on Steroids: SGX-Based High Performance BFT | EuroSys'17

拜占庭容错（BFT）协议需要3f+1个replica来保证网络的运行。将节点运行在SGX中，节点不再可能出现恶意，只有宕机的可能，由此将replica的数量从3f+1变为2f+1，由此降低整体资源要求，提高共识效率。

### PANOPLY: Low-TCB Linux Applications with SGX Enclaves | NDSS'17

和在SGX中实现LibOS的方案不同，PANOPLY在SGX中以LibC为切面，提供了大量C接口来支持Linux程序在SGX中的移植。
虽然TCB代码量少了，但是暴露的接口却多了。大量的接口也导致性能损失，安全上无法缺乏保证。

### SGXIO: Generic Trusted I/O Path for Intel SGX | CODASPY'17

SGX作为TEE中的一员，却缺少像ARM TrustZone这样的可信IO支持。
SGXIO是第一篇探讨SGX可信路径的工作。借助可信的Hypervisor在不可信的OS上提供可信路径支持。
唯一感到别扭的是HV本是被排除在SGX的TCB之外的，这里复合了传统地引入可信HV的工作。

### Secure Live Migration of SGX Enclaves on Untrusted Cloud | DSN'17

SGX不允许Hypervisor查看其内存页，因此云也就无法完成传统的EPC迁移工作。
为此需要在SGX的库中增加支持，利用类似检查点的技术由SGX程序自行停下并主动完成迁移工作。
这里的关键技术点是如何保证迁移时不会遇到fork攻击。

### Regaining Lost Cycles with HotCalls: A Fast Interface for SGX Secure Enclaves | ISCA'17

本文利用已被SCONE和Eleos使用的多核异步技术来避免SGX模式切换导致的性能问题。
文章移植了3个经典网络应用，找出其模式切换最多的系统调用并替换为本文提供的异步接口，加速效果明显。

### Glamdring: Automatic Application Partitioning for Intel SGX | ATC'17

如果不希望使用LibOS这样的大TCB方式，则需要开发者手动指明哪些部分需要保护，并手工进行分拆。对于现有应用而言，手工分拆费力不讨好，而且也很难保证不出错，即秘密不会意外泄露出去。
Glamdring主要利用了程序分析手段，借助数据流分析和程序前向切片的思路对保证工具链拆分的正确性和安全性。

### Enhancing Security and Privacy of Tor's Ecosystem by Using Trusted Execution Environments | NSDI'17

Tor是洋葱网络的称呼。洋葱网络本身是去中心的，文件夹节点暴露在野外环境中，可能被恶意者控制并试图摧毁洋葱的匿名保证。
本文主要写的是Tor在SGX的移植工作。受保护的Tor节点可以进一步增强洋葱网络的安全性。

### Komodo: Using verification to disentangle secure-enclave hardware from software | SOSP'17

SGX是由CPU硬件提供的安全保证。一旦硬件出现bugs，想要打补丁是很难的。
Komodo结合了TEE和形式化验证，在ARM TZ中模拟了SGX的基本能力，并用形式化保证其正确性。一旦SGX需要打补丁或升级，无需更换硬件，更换形式化组件即可。

### X-Search: Revisiting Private Web Search using Intel SGX | Middleware'17

本文尝试将SGX用于隐私网页搜索，实现类似duckduckgo的匿名功能，但无需相信隐私服务提供者。
X-Search在云上搭建一SGX代理，此时已实现类似VPN的匿名化目的。为了进一步隐藏搜索意图，X-Search利用历史搜索进行混淆，因此有一定精度缺失。

### Graphene-SGX: A Practical Library OS for Unmodified Applications on SGX | ATC'17

本文将Graphene移植入SGX中，实现针对Linux系统上的SGX LibOS支持，并进行了开源，得到Intel和Golem的支持。
该项目在SGX领域的影响是不小的。

### VAULT: Reducing Paging Overheads in SGX with Efficient Integrity Verification Structures | ASPLOS'18

SGX的EPC物理资源大小受限的根本原因在于内存完整性的支持。
Merkle Hash Tree的深度太大会导致更多EPC metadata的占用，也会加剧每次EPC更新的开销，因为MHT的要求是每当某叶子节点更新后，相关路径也必须跟着更新。
本文提出了新的Bonsai Merkle Tree构造，极大程度地缓解了这一困局。

### EnclaveDB: A Secure Database using SGX | S&P'18

本文将整个内存数据库放入SGX中进行保护。
关键技术点在于如何支持日志、事务、并发等数据库关键技术点。

### Oblix: An Efﬁcient Oblivious Search Index | S&P'18

SGX容许将ORAM Client放到公有云上，减少ORAM C/S之间的round-trip开销。
但是SGX本身有内存模式泄露问题，Oblix由此考虑的是如何将ORAM Client也保证Oblivious。
本文实现的是双Oblivious数据结构，该结构的开销并不像想象中那么大。

### ZeroTrace: Oblivious Memory Primitives from Intel SGX | NDSS'18

本文将Path ORAM的算法移植入了SGX。为了实现类似Oblix的效果，本文采用对客户端采取了递归使用ORAM的思路，性能上有一定开销。

### OBLIVIATE: A Data Oblivious Filesystem for Intel SGX | NDSS'18

本文实现了一个应用了ORAM的用户态文件系统，服务于多个SGX实例（类似微内核）。

### ndBox: Scalable Middlebox Functions Using Client-Side Trusted Execution | DSN'18

SGX通常用于服务端，防范不可信的云。本文反其道行之，用在客户端。从而利用客户端的数量规模实现可扩展的目的。

### LibSEAL: Revealing Service Integrity Violations Using Trusted Execution | EuroSys'18

本文在云上实现一个服务，检查文件提交后检索的完整性。

### PESOS: Policy Enhanced Secure Object Store | EuroSys'18

本文在云上利用SGX CPU和智能存储间构建可信信道，实现基于策略的安全存储。

### Migrating SGX Enclaves with Persistent State | DSN'18

之前的SGX迁移只考虑了EPC内存状态，本文额外考虑了加密后的存储资源，以及时钟和可信计数器等外部状态。

### SafeBricks: Shielding Network Functions in the Cloud | NSDI'18

本文将NFV、DPDK、Rust等结合在一起，实现一套安全的MiddleBox。

### DelegaTEE: Brokered Delegation Using Trusted Execution Environments | USENIX Security'18

SGX提供了可信的共享沙盒，可以将个人的账号密码保存其中，允许第三方通过授权完成安全共享服务。

### ShieldStore: Shielded In-memory Key-value Storage with SGX | EuroSys'19

借助SGX实现安全的KV，对KV的内存管理引擎做了修改，类似Eleos这样避免性能开销。

### Slalom: Fast, Verifiable and Private Execution of Neural Networks in Trusted Hardware | ICLR'19

对神经网络模型进行拆分，将线性部分放到SGX外进行计算，并对线性结果进行验证。
这种针对workload的属性进行拆分，将容易验证的部分防止外部，难以验证的部分借助SGX原生算力来保证计算结果正确性的思路值得学习！

### Clemmys: Towards Secure Remote Execution in FaaS | SysTor'19

考虑到Serverless需要快速启动的特点，本文利用SGX2的指令实现Enclave的快启动。

### BITE: Bitcoin Lightweight Client Privacy using Trusted Execution | SEC'19

将SGX用在比特币的转账查询上，借助ORAM防止SGX机器知道结果，用SGX保证正确性和隐私性。

### Towards Memory Safe Enclave Programming with Rust-SGX | CCS'19

Baidu Rust SDK提供了Rust版本的SGX SDK。Rust语言很大程度缓解了C/C++的内存安全问题。
该项目的开源是SGX代码安全的重要里程碑。

### Plundervolt: Software-based Fault Injection Attacks against Intel SGX | Oakland '20

事实证明，对电压的调节能导致CPU出错，SGX的执行也不例外。
因为OS掌握在不可信人手中，因此可以通过控制电压导致CPU执行出错，从而让保护SGX的秘密泄露出来。

### ObliDB: Oblivious Query Processing using Secure Enclaves | VLDB'19

本文对数据库的index部分进行了替换，借助SGX保证隐私查询的泄露面最小化。

### CoSMIX: A Compiler-based System for Secure Memory Instrumentation and Execution in Enclaves | USENIX ATC'19

本文借助编译器技术，对代码的堆分配进行插桩，实现类似Eleos这种由软件实现的加密换页效果。

### Occlum: Secure and Efficient Multitasking Inside a Single Enclave of Intel SGX | ASPLOS'20

本文利用MPX在SGX内部实现隔离，从而实现多任务快速fork和高效共享的效果。

### MPTEE: Bringing Flexible and Efficient Memory Protection to Intel SGX | EuroSys'20

MPX还能解决SGX1代中内存权限运行时无法改变的问题。本文的设计十分巧妙，值得阅读。

### Autarky: Closing controlled channels with self-paging enclaves | EuroSys'20

SGX的内存模式可见导致的隐私泄露问题十分突出。
本文尝试对硬件做出改变，允许SGX程序自己换页，再借助一些类似ORAM的措施来保证换页行为不泄露信息。
