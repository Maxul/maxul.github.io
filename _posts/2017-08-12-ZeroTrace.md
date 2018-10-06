**题目： ZeroTrace : Oblivious Memory Primitives from Intel SGX**

**作者： Sajin Sasy, Sergey Gorbunov, Christopher W. Fletcher**

**单位： University of Waterloo and Nvidia**

**出版： NDSS 2018**

ORAM内存协议保证攻击者只能得知内存访问的频次，但对于访问模式（如读写地址的规律）无法学习，由此保证数据整体的机密性。

# 解决问题：
1. 我们身处一个大量使用应用密码学和可信硬件系统的云时代；
2. Oblivious memory primitives (ORAM) 效率不高，而Intel SGX技术则被证明存在各式各样基于软件的旁路攻击；
3. 我们将二者技术融合起来，在可信硬件上构建ORAM，优势互补，缺点消失：一方面ORAM的加密效率提升，另一方面可以抵制各种基于软件的旁路攻击。

技术的核心是块级别的内存控制器，其实现Oblivious算法。

# 挑战：
1. SGX技术本身要求开发者自行定义可信和不可信部分，对简单应用没问题，对复杂大型程序而言不实际；
2. 一些工作旨在提供不经修改的程序运行在enclave的能力，如pthread、libc、contaner等。

本工作提供一个静态库，支持对应用程序数据结构的细粒度的ORAM内存访问模式隐藏。

# 贡献点：
1. 第一个在安全硬件上实现ORAM内存控制器的工作；
2. 设计并实现了ZeroTrace，一个用于隐藏数据结构访问模式的静态库；
3. 对1TB数据集的“隐藏”读写为267ms（4KB粒度）；对“即插即用”数据结构，80MB数组的8字节读写为1.2ms。


# 适用模型：
客户端希望将计算和存储外包给具备SGX安全特性的远程服务器。
* 内容包括但不限于：图片、电影和文档；
* 客户端：任何移动或本地设备；
* 服务器：亚马逊AWS、微软Azure和谷歌cloud。


# 威胁模型：
1. CPU和SGX机制可信，之外均不可信；
2. ORAM控制器实现正确，客户端可信；
3. OS可以借助系统调用、cache访问和缺页中断对enclave内的数据进行窥探；
4. OS可以随时中断enclave的执行，此时SGX状态消失，用户数据存在丢失风险（本文提供容错机制）；
5. 不考虑硬件攻击（如能耗分析、EM  emissions）、硬件后门和DoS攻击。


# 设计：
* 对内存的load、page、execute进行形式化阐释，对ORAM算法的复杂度进行说明。
* 为减小TCB和使安全存储（IO操作）成为可能，ZeroTrace没有把所有ORAM逻辑放在Enclave中，而是拆分成了前后端两部分。
* 使用CMOV汇编进行数据拷贝的加速。


# 实验：
* 几乎清一色的实验是在戴尔品牌的机器上实现的。
* i5 6500，4核；128MB EPC；64GB DRAM，西部数据500GB HDD。
* 4000LoC，1800LoC在Enclave中。


# 思考：
* DBLP上关于内存和高速缓存Oblivious的信息安全工作不下百篇。
* ORAM技术是一个老话题了，本工作赋予了该技术新的生命。
