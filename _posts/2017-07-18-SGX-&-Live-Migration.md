**题目： Secure Live Migration of SGX Enclaves on Untrusted Cloud**

**作者： Jinyu Gu, Zhichao Hua, Yubin Xia, Haibo Chen, Binyu Zang, Haibing Guan, Jinming Li**

**单位： IPADS & Huawei Technologies**

**出版： DSN '17**

#解决问题：
* 由于SGX禁止VMM/OS访问enclave的运行状态，使得传统的动态迁移无法实现。


#阅读前思考：
之前所有的保护工作都假设，enclave不会主动泄漏自己的信息。而本工作独辟蹊径，由于外界无法直接访问内部，因而在enclave内部创建控制线程，将迁移所需的信息转移出去(encrpt and checksum & dump)。对于软件不可访问数据，如CSSA (Current State Save Area)，则采用推测(infer)的手段。

背景介绍部分描述了作者的行文思路。

作者指出，他们的SDK在Intel发布的Linux SDK之前。

笔者知道，该团队对Qemu的研究很深，有能力修改Qemu，使其支持SGX。

#贡献：
第一个针对包含由enclave安全域的VM的动态迁移工作。
1. 对包含enclave的VM的实现挑战和潜在攻击的研究；
2. 对指出的安全问题进行了解决；
3. 对安全分析和性能评估进行了详细说明，对未来SGX的设计提出了建议。


#挑战：
1. Enclave的状态信息对Hypervisor不可见，而且是加密的；
2. VMM和OS不可信，会影响一致性；
3. “弹性计算”云允许克隆VM和借助检查点回滚，存在**fork attacks**和**rollback attacks**。


#目标：
* 状态保密性：迁移期间状态不可泄漏；
* 状态完整性;
* 状态一致性；
* 状态保持性：不可回滚；
* 单实例属性：只有一个迁移实例允许恢复运行restore；
* 小TCB。


#威胁模型：
* 同云假设。
* 拒绝接受迁移实例的DoS攻击不考虑。


#系统架构：
1. SDK自动在每个enclave实例内插入一个控制线程(CT)；
2. 二阶段检查点机制：在source机器上，CT负责生成检查点；1) target首先生成一个躯壳，2) source负责进行远程验证，成功则创建安全信道，传递密钥（检查点的钥匙）；3) target借助检查点恢复内存；4) target检查CSSA，即enclave内部状态”快照“的一致性。5) 恢复执行。


#实现：
编码量：50 LoC in Qemu, 1584 LoC in KVM and 5798 LoC (except the source code of libc inside enclave) in the guest VM.
EPC的管理：
* VMM在启动初期将所有EPC映射为VM可访问的虚拟地址；
* 添加新的hypercall，支持guest VM直接访问EPC地址和大小；
* guest VM用完EPC前不会触发EPT (Extended Page Table) violations，可提升性能；
* VMExit Inside an Enclave：Enclave模式下，发生VMExit会在VMCS (Virtual Machine Control Structure)的会设置“Enclave Interruption”比特位。

评注：该论文的提到的vmexit和enclave，可见Intel的设计中已经考虑了虚拟化技术和可信硬件的整合。这证明类dune+enclave的方案是可行的。目前估计，驱动将复用kvm和dune，前端将是二者功能模块的结合。

虽然在实现vmOS的时候学习过Qemu，但KVM利用的VMX技术缺乏研习。对VMCS的结构信息初步查看，和TI的C6000系列寄存器设计很类似：一个是Host+Guest，一个是Parent+Child，可见TI的硬件设计者在设计ISA时可能参考了一部分思想。（猜测）

#给SGX的建议：
1. 添加EPUTKEY，支持多CPU共享一个密钥（但据我对Intel材料的认知，这个密钥生成自CPU Package，每次启动都不一样，不是一个固定属性）；
2. 添加EMIGRATE，使enclave不可执行，避免一致性问题；
3. 添加ESWPOUT，允许EPC换出，但需要有签名密钥的MAC维持（密码学。。。）
4. 类似的，在target上，有ESWPIN等。


#性能评测：
To migrate a VM with 64 enclaves running inside, the total time of migration grows by 4.7%, and the downtime increases by only 3 milliseconds.
（negligible这点可以理解，因为VM本身就有很大的performance degradation。）

