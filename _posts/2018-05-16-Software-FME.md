**题目： No Sugar but All the Taste! Memory Encryption Without Architectural Support**

**作者： Panagiotis Papadopoulos, Giorgos Vasiliadis, Giorgos Christou, Evangelos P. Markatos, Sotiris Ioannidis**

**单位： FORTH-ICS, Greece**

**出版： ESORICS 2017**

# 解决问题：

1.    对于可能物理接触设备的攻击者而言，类似密码、隐私数据这样的数据可以直接从主存中获取。对这些数据的保护是个重要问题。
2.    本文探索如何在无体系结构支持下实现内存加密，同时开销合理。
3.    为此**第一次提出纯软实现方案，保证主存数据永远处于加密状态**。
4.    方案基于现有物理硬件，对应用透明。可以选择为全加密还是选择性加密。


# 背景：

1.    对于设备偷窃、物理攻击、工业间谍等，全盘加密的保护十分广泛。我们称之为cold data保护。
2.    对于运行在内存中的hot data，明文数据的危险性不言而喻。研究表明，40%的服务器和桌面设备在无人看管时使用休眠模式，session keys, passwords, HTTP cookies, SSL key pairs等数据岌岌可危。
3.    现有的内存方案都是基于体系结构完成的，因而兼容性不强。
4.    本文每次在每次创建新进程都使用新的128位AES密钥，因此有效阻止key guessing attacks猜密钥攻击。使用AESNI加速。


# 贡献点：

1.    第一个纯软的内存加密方案。
2.    提供一个库帮助用户创建加密内存segment的需求。
3.    使用静态和动态插桩方案进行微观和具体应用宏观量化评估。


# 威胁模型：

1.    冷重启攻击：冰箱冷冻RAM电子，或迅速迁移到别的读取设备上；
2.    DMA攻击：Firewire, PCI, PCI Express和Thunderbolt。
3.    不考虑：软件漏洞（控制流劫持而主动泄露），旁路（时间或能源消耗）， Advanced Persistent Threats (APTs)（商业间谍、政府或军队后门）。
4.    假设CPU之外的设备都不可信，内核可信（本文认为控制内核者可以做更多坏事，单纯偷内存数据太小看了）。


# 可选方案：

1.    静态方法：性能更好，但需要对所有共享库都做插桩；
2.    动态方法：适合JIT、obfuscated or self-modifying code，因其灵活性，本文选择它。


#设计：

1.    AES属于块加密，至少需一块128位（两个xmm寄存器，一个load一个store）。DRAM->SIMD Regs->GPRs。为了避免访存模式泄露隐私，使用流模式（CTR）代替块模式。此时在随机读写上速度可以提高。引入per-session random nonce and per-block counter。
2.    系统调用：从用户态传入内核态的参数仍加密。为避免模式切换开销，使用vDSO实现加密指令服务。
3.    signal和setjmp/longjmp：对于控制流的突然变化（异常），保存上下文不允许寄存器中明文直接存储到内存中，因此修改内核实现，保证加密后再写入。
4.    线程切换：保证线程拥有寄存器在切换时被加密。对于XMM寄存器，FXSAVE前加密，FXRSTOR后解密。
5.    Selective memory encryption (SME)：FME开销太大，为此引入选择性加密方案。#pragma接口只能保护静态数据，对heap无能为力。为此引入secure memory allocator，提供s_malloc，对128位倍数的数据进行内存加密。
6.    以上方案只能保证保密性，对于完整性则使用IOMMU拒绝DMA破坏内存。
7.    密钥管理：本方案对每个进程有个密钥，存放在PCB数据结构中。每个密钥都被加密后再存放，而主密钥则存放按debug寄存器中dr0-dr3（类似Tresor）。
8.    Bootstrapping：开机时使用0核执行RDRAND生成主密钥，存在debug寄存器中，所有核spin（自旋）MTRR寄存器忙等0核的主密钥，此时主密钥一致（主钥一样有助于线程核间迁移）。FDE不使用该密钥，对于交换空间的数据使用FDE的密钥，否则休眠唤醒后swapped-in的数据无法复原。
9.    动态插桩：使用PIN运行程序，对每次内存访问都做检查。PIN支持对用户程序的每条指令都做检查和修改。主要关注load/store指令。


# 性能评测：

1.    SPEC CPU2006：PIN比原生下降6倍，PIn+FME（软）比原生下降10倍。开销来自加解密带宽、被测对象的访存策略（是否有局部性）和prefetch机制。
2.    SQLITE3：随row的增加而线性增长，从图中看，表插入操作的开销大概是2倍左右。
3.    Lighttpd web server：使用另一台物理机器进行测试，FME引入开销几乎可以不计，0.4%，**被网络延迟所掩盖**。对于高频HTTP请求（100/1000 Mbps）开销在17%左右。
4.    SME：对于含10%敏感数据的程序而言，微观评测引入24.90倍开销。宏观用Lighttpd和WolfSSL，将SSL握手的部分进行SME，ab评测结果表明：引入了27%的延迟。
5.    静态插桩：对 SPEC suite翻译后的汇编代码做静态插桩，无需PIN的动态翻译技术（也就没有的翻译延迟）。


# 局限：

1.    因为FME整体加密对象为一个进程，由于进程间密钥不一致，因此进程间通信的共享内存是个问题。因此需要将粒度缩减到segment上。
2.    对于和DMA设备的数据交换必须在DMA内存区域是明文（如GPU显示），解决方案有待继续研究。


# 相关工作：

1.    CaSE (S&P '16)：使用TZ、SoC特性和cache locking在智能手机和手持平板上实现全内存加密。
2.    Sentry (ASPLOS '15)： 使用ARM SoC存储代替DRAM保护敏感信息，**只在息屏情况下执行该操作**。
3.    AESSE：用SSE保存密钥，但无法给多媒体程序（如OpenSL）使用。TRESOR换成了dr调试寄存器，方法被公认为极佳选择。
4.    PivateCore’s vCage：商业级别产品，使用cache方案保护敏感数据，引入可以VMM。
5.    BitArmor：FDE硬件级方案。SGX只能保护用户态部分数据，尚不支持动态加载；但能作为本方案的保护密钥的互补技术。


# 体会：

1.    这是一篇写作很好的文章，通俗易懂。
2.    本文引入可信内核，FME的对象为用户进程。动态方式是OS启动PIN作为JIT，拦截内存访问；静态为代码插桩。除网络外，文件系统、计算密集型开销太多。**SME选择内存加密方案值得借鉴，其本质和SGX一样，需要人工裁定**。
3.    **本文Lighttpd几乎零开销的实验再次证明：网络IO自身的开销仍是现实通信瓶颈的主导，模式切换、FME都无法与之比肩**。


# 思考：

1.    内存加密的常见方案是依赖特权软件。我们是否可用用户态SGX来实现，此时先天就有EPC可以存放密钥，进而实现全内存加密。一个方案是建立足够多的Enclave，那么内存就会被page out的EPC Shadow所填充，但只能对用户态内存加密。另一种是结合SGX和Eleos（软件换页方式），这个方案也可以用在内核态，只不过引入SGX Crypto Engine而已。
2.    我目前的方案：使用Dune，将所有EPT内存都清除Present位，此时sandbox的程序访存会触发缺页中断，进而得知其每次访问的地址。我们在中断处理函数进行加解密操作。
