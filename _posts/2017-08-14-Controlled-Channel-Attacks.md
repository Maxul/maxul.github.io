**题目： Controlled-Channel Attacks: Deterministic Side Channels for Untrusted Operating Systems**

**作者： Yuanzhong Xu, Weidong Cui, Marcus Peinado**

**单位： The University of Texas at Austin and Microsoft Research**

**出版： IEEE Symposium on Security and Privacy 2015**

Yuanzhong Xu在微软研究院实习期间习作。
SGX旁路攻击**第一杀**。种子文章。

# 解决问题：

*    控制信道攻击是一种有效的攻击手段。
*    在云环境中，由于上下文切换，缺页中断、TLB刷新、中断和异常存在大量噪声，控制信道攻击难度很大。
*    SGX旨在将OS和VMM移除出TCB。但在SGX技术实现中，OS仍旧负责内存管理、存储和网络，这给了OS进行Iago成功攻击增加了可能。
*    SGX的EPC缺页中断在控制信道不存在噪声，本工作成功对Haven和InkTag进行了有效攻击，能抽取文本和图像数据。


# 挑战：

1.    4KB大小的页粒度。
2.    每次访问页都会触发中断，由此跟踪页访问情况。会对应用产生巨大性能开销。


# 贡献点：

1.    缺页攻击值得引起开发者注意。
2.    对广泛使用的第三方库进行成功攻击。
3.    Haven和InkTag有效攻击。



# 假设：

*    二进制镜像已知，如广泛使用的libjpeg、FreeType font rendering engine、Hunspell spell checker。


# 攻击模型：

1.    OS负责内存管理；
2.    应用使用已知的第三方代码。


# 先验知识：
从输入相关的控制流和数据流入手，举例如下：
```
void CountLogin (GENDER s) {
    if (s == MALE) {
        gMaleCount ++;
    } else {
        gFemaleCount ++;
    }
}
```
1.    此时Enclave的页访问模式会根据OS的输入而发生变化。
2.    跟踪Enclave实际访问的页，猜测其行为。
3.    该攻击对ASLR一样有效，可以很容易得到动态加载的地址（第一次缺页中断发生由loader触发）。文章直接谈到，**ROP攻击的可能性（种子想法）**。


如何猜测：
以Hunspell为例，由于其哈希算法的确定性，即每次读取单词的内存访问路径和单词拼写是一一对应的，由此逆向判断Enclave的数据是容易的。


# 攻击效果：

*    Hunspell精确度最高可达96%，FreeType最高为100%。
*    针对Libjpeg 9a库，信息泄露效果如下：


# 对策：

*    使用隐藏访问模式（Oblivious），可以借助编译器或运行时来完成。已分别由如下两篇完成：

    1. 工具链：A Compiler and Verifier for Page Access Oblivious Computation. ESEC/FSE 2017
    2. 运行时：ZeroTrace : Oblivious Memory Primitives from Intel SGX. IACR 2017

*    更细粒度的ASLR：SGX-Shield: Enabling Address Space Layout Randomization for SGX Programs. NDSS 2017
*    Self-paging：Self-Paging in the Nemesis Operating System. OSDI 1999



# 局限：

*    依赖于对二进制的先验知识：猜测对方二进制镜像使用了什么库和算法，根据已有的经验进行信息抽取。



# 体会：

1.    这篇可以认为是SGX攻防的种子文章，启发了不少工作。
2.    Dark ROP是针对Buggy Enclave的Iago攻击。结合本文，可以总结出本文和Dark ROP的局限，必须依赖于二进制镜像的可获取性。但其实这个局限也容易克服，用户发布在云上的进程实例要么来自于存储，要么来自于网络。
