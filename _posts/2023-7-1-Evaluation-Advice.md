# 系统实验有哪些坑？

## 实验的目的

* 比较不同系统的相同维度，体现不同系统之间的性能表现差异
* 比较同一系统的不同版本与改进，在同等环境条件下的性能差异

## 实验的意义

* 理解系统极限（或局限、瓶颈），评估系统容量
* 验证系统假设，确认系统运行结果是否符合预期
* 重现系统异常行为，找出原因并解决
* 规划业务增长，评估扩容需求，降低运行风险

## 实验的要求

1. 可观测性：测试对象可具体量化和比较
2. 可重复性：相同条件下测试结果可重现
3. 可对比性：测试前置条件设计公平合理

## 实验的设计

### 挑选合适的基准测试套件

1. 提出问题并明确目标：
   * 是使用工业界标准化的基准测试工具，还是设计专用的/定制化的测试用例？后者须给出可信服的具体理由。
   * 需要测试哪些指标（metrics）：吞吐量？延迟（响应时间）？资源利用率？并发性？可扩展性？
2. 建立测试结果的规范化文档：
   * 对每轮测试都进行详尽的记录和归档
   * CSV电子表格、自定义数据库，适合自己的都可以
   注：基准测试（benchmark）：包含了一组合适的workloads和潜在的dataset，以及对应的评估标准

### 收集系统状态

1. 用自动化脚本收集系统运行状态，并记录在案：
   * CPU使用率、磁盘I/O、网络流量
   * 网络拓扑、客户端并发程度
2. 是否要屏蔽无关噪声？
   * 中断、调度是否导致结果不确定？
   * 其他应用负载的运行干扰？
   * 初始化配置是否一致？
   * 资源分配是否合理？

### 获取准确结果

1. 确保测试的可重复性：
   * 每次重新测试前系统的状态是否一致？
   * 预热时间是否足够？测试时间是否足以消除方差(\<1%)？
   * 有哪些干扰要素：内存碎片？一致性检查？垃圾回收？网络拥塞？
   * 负载是否来自真实数据，还是自定义的模拟，模拟的分布是否符合客观现实？
2. “控制变量法”：
   * 每轮实验修改的变量类型和个数应尽可能少，降低实验分析的复杂度
   * 不同变量间是否存在依赖或相关性？
   * 系统的缓存是否会干扰实验结果？

### 正确作图和结果分析

1. 和其他系统进行基线（baseline）比较：
   * 给出其他系统的性能表现
   * 是否给出绝对值，绝对值是否符合经验？正则化可对数据做一定脱敏
   * 线性坐标还是对数坐标更合适？
2. 结果可视化呈现：
   * 区间选择是否充分说明系统的最大上限？如最初10个点可以线性扩展，但没有给出系统到一定程度出现的瓶颈
   * 平均值、峰值比较是否附带方差？
3. 异常点：
   * 成功vs失败等不同事务的占比统计
   * 异常情况是否可重现？

### 结果的文字说明

* 文字说明应当尽可能完整细致，有具体的性能剖析，以及有说服力的原因分析：

> *As an experiment to investigate the performance of the resulting TCP/IP implementation ... the **11/750 is CPU saturated**, but the **11/780 has about 30% idle time**. The time spent in the system processing the data is spread out among handling for the **Ethernet (20%), IP packet processing (10%), TCP processing (30%), checksumming (25%), and user system call handling (15%)**, with **no single part** of the handling dominating the time in the system.* from Bill Joy, 1981, TCP-IP Digest, Vol 1 #6.


### 实验的留档
- 越来越多的系统会议鼓励论文的实验过程和结果是可复现的，该过程被称为 Artifact Evaluation (AE)。 
- 实验的留档有助于AE的准备，因此平时就应养成良好习惯。
- 推荐MIT做法，如在[AIFM@OSDI'20论文的开源代码](https://github.com/aifm-sys/aifm)里，他们对论文每个图片都保留了测试：
  `AIFM/aifm/exp/figurex/...`，里面包含了代码、配置文件以及自动化测试脚本。 
  **该步骤显然可以在准备论文实验的时候一起做好**。这样，无论是别人来复现你的代码，还是准备AE都会事半功倍。

## 注意事项

1. 现成的benchmark可能不对结果的正确性进行检查。评估网络传输或文件存储性能时，**请务必将数据读回并检验内容正确性，验证实验结果确实如预期进行**。
2. 每次测试尽量不要使用同一份数据。可以在数据中加入时间戳或其他ID，防止反复读取的是缓存数据。可利用标准差，找出潜在的缓存因素。
3. 随机化数据集使用的次序，有助于识别测试组合间的关联。可从不同方向遍历数据集合来实现。
4. 不要只使用常规的2次幂步长。2n-1、2n+1的点可能给出个例（corner cases）的性能特征。
5. 确保每次比较的对象在尽可能多的相同运行状态下运行。如内存分配是否处在连续的物理内存上。
6. 多次运行下应始终检查标准差。对>1%标准差的结果进行核查。
7. 在某些情况下，忽略统计学上的最大值和最小值是合理的，但是应在论文和报告中应给出完整论述和做法。
8. 留出足够长的时间进行预热，用尽可能多的迭代来消除噪声。
9. 将被测对象放到一个独立的函数中单独计时，如系统调用。
10. 微观实验（microbenchmark）要记得消除循环调用的开销：在未开启编译器优化的条件下将调用替换为nop，先评估出函数调用或循环的开销。

## 常见误区

1. **在基准测试集上做手脚**：
   * 只测试了部分子集，却缺乏合理性解释。随机选择子集和手动选择是不一样的，前者更容易被接受。
   * 没有指出潜在的其他性能代价：在给出的基准测试上具有性能优势，但在其他维度导致了新的代价却缄口不言。
   * 参数范围选择不合理：实验区间不足以反映系统的完整特征。
2. **错误处理基准测试结果**：
   * 用微观基准代替整体性能：再全面的微观评测也不能代表系统整体性能。讨论系统真实性能，务必使用宏观基准。
   * 将部分维度当成系统的整体特性：例如将吞吐量变化当作整体开销，而忽视系统修改对延迟造成的影响。
3. **在数据呈现上“做文章”**：
   * 用百分比增长混淆实际代价：开销从6%增加到13%，代价并非只增长了7%，而是加倍了！
   * 没有将原系统作为基线：计算相对性能开销时，原系统性能表现应作为分母。
     例如：延迟从60s降为80s，性能下降不是1-60/80=25%，而是80/60-1=33%
   * 缺少多次实验：计算机系统在运行时可能因为抖动可能引入噪声，因此需要测试多组数据。选择数据时应该把warmup和steady两个阶段分开，以保证结果是在系统行为相对确定的情况下获得。
   * 缺乏方差说明：计算平均值时应带方差。建议使用t检验说明显著性。
   * 统计学参数：几何平均适合于不同区间的比较，调和平均适合于比较比率时。当数据分布有异常值时，应该给出中位数。
4. **测试集本身存在问题**：
   * 仿真实验：在仿真环境下支持现有基准套件是具有挑战的，通常需要修改测试程序本身。基准测试自身有一定的假设，而仿真本身也包含了假设。应检查仿真基准测试是否仍具有代表性。
   * 测试集选择有误：比如用SPEC CPU来测试多核的可扩展性，而该测试集缺乏对底层通信的负载；或使用CPU密集型的测试用例来测试网络开销。
5. **实验结果的不合理比较**：
   * 缺乏基准结果说明：将两个不同解决方案的论文结果直接对比，而缺乏原系统基准结果的数据。
   * 相关工作的测试环境是否一致：应尽可能描述系统配置的所有参数，并对相关工作的真实测试结果与发表数据的不一致进行说明。
   * 只和自己对比结果：对自己的系统做了改进，并只与自己之前的结果做对比。应加入现有其他系统（state-of-the-art）的最新测试结果进行比较。
6. **测试信息呈现不全面**：
   * 基准测试环境说明：为保证可重复性，应尽可能将所有可能影响结果的参数进行说明。一般包括处理器架构、核心数、主频和内存大小。网络最好能说明网卡支持的最大带宽。内存测试应包括缓存大小和级联性。软件部分应包括系统内核版本号，以及可能的工具链版本。
   * 只给相对测试结果：相对测试结果的数据缺乏足够信息量，应给出测试结果的绝对值。例如：如果说A系统是B系统开销的2倍，在开销绝对值为10倍和20倍条件下两个系统均是不实用的，但开销从0.1%到0.2%反而更容易被接受。

## 其他补充

1. 实验部分不仅要说明效果（what's been achieved），还要解释是怎么做到这个效果的（how to achieve it），这些效果如何说明前面设计的有效性。
2. 要避免“言过其实”：比如只适用于一部分用例/情况，但声称对所有情况均适用；又或者其实需要不少人力介入，但号称该过程能全自动化。
3. 要主动澄清自己的局限和不足，避免后续研究走弯路。理由：Progress comes in steps, rarely in leaps, and we need those steps to be solid and clearly defined.
4. 相关的baseline可能不是为当前场景服务的，要首先优化该baseline，再进行比较。好的实验可以说明设计的优越性：哪怕相关工作尽力了，也无法超越你。
5. 一定要小心比较配置的环境和参数。比如相关工作的-O0和-O3性能差距较大，要合理公平比较。
6. 某些因素的相关性需要进一步说明。例如，缓存不命中的下降，和端到端延迟的下降，或者功耗的降低是否强相关，是否存在其他因素也导致这一现象出现？
7. 测试结果是否完整反映出系统带来的影响？**比如优化，是对哪些负载效果更好；或者开销，是否有选择性地隐瞒了某些用例**。只要能解释清楚，并想出针对那些目前还不行的情况的一种潜在优化方法（如cite其他论文），都是能被大家接受的。
8. 画图时要小心：使用对数坐标，或者坐标轴有截断时，一定要主动说明，避免误导读者。

#### 参考资料
1. [Systems Benchmarking Crimes - Gernot Heiser](https://www.cse.unsw.edu.au/~gernot/benchmarking-crimes.html)
2. [SIGPLAN Empirical Evaluation Checklist](http://sigplan.org/Resources/EmpiricalEvaluation/)