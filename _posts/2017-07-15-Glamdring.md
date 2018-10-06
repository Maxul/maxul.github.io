**题目： Glamdring: Automatic Application Partitioning for Intel SGX**
**作者： Joshua Lind, Christian Priebe, Divya Muthukumaran, Dan O'Keeffe, Pierre-Louis Aublin, Florian Kelbert, Tobias Reiher, David Goltzsche, David M. Eyers, Rüdiger Kapitza, Christof Fetzer, Peter R. Pietzuch**
**单位： Imperial College London, TU Dresden, TU Braunschweig, University of Otago, TU Braunschweig**
**出版： USENIX Annual Technical Conference 2017**


#导读：
* 该工作借助**静态分析**手段，根据用户标记的隐私数据**自动**找出安全相关代码。
（笔者注：这本是数据流分析中常见做法，本工作将其与SGX结合，找到新的用武之地。）


#解决问题：
1. 之前使用CPU硬件保护数据中心隐私数据的工作的TCB都太大；
2. 作者认为，之前有些工作（VC3、Moat）对enclave内的代码和外界交互做了太多限制，不通用不方便（limits the applicability of trusted execution mechanisms for arbitrary applications）；
3. 作者洞见（observation），**人工找出的安全相关代码并不见得是最小集**，旨在最小化TCB；
4. 笔者认为，根据Intel给出的SDK和工具链，需要人工标注可信和不可信接口，对现有大型程序的移植是个难题。本工作的创新和贡献点很棒。


#方案：
见原文图2。

* 用户标注好自己要保护的数据，Glamdring自动将程序分成可信/不可信两组：
1. 根据**数据流分析**找出相关函数；
2. 使用**逆向切片（backward slicing）**保证数据完整性；
3. 基于 LLVM/Clang，Glamdring将找到的安全相关函数放入enclave中，自动生成entry/exit点，添加内存检查代码和运行时检查（不变式），用加密保证安全，该步会增加TCB。

注：静态分析得出程序依赖图，人工标注类似污点分析。


#威胁模型：
* 和之前针对不可信云的工作假设一致。


设计：
见原文图1。

1. 第一类是在Enclave内的全系统级别：Haven、SCONE以及Graphene，包含了安全无关代码，TCB太大；另外，SGX自身的限制无法克服，SCONE容器无法支持fork系统调用。
2. 第二类是*预定义好交互接口*的，TCB很小，如VC3。但使用场景有限，不及第一类通用。
3. 第三类是想针对不同应用或使用场景进行合理划分，同时省去人工编写EDL（Intel工具链提供的针对安全接口声明的DSL）的成本。


#攻击场景：
这里主要考虑静态工具作用：
1. 接口攻击：函数不变式等；
2. Enclave调用重排攻击：数据流与控制流依赖关系；
3. Iago攻击：返回值不变式；
4. 重放攻击：加密与版本校验（freshness）；
5. Enclave代码漏洞：静态检查与预防。


#实验：
1. Memcached
2. LibreSSL
3. Digital Bitbox bitcoin wallet
* TCB增加：22%–40%
* 开销引入： 0.3×–0.8×


#体会：
* 强烈推荐给PL组精读。思路和行文非常值得借鉴。文中给出不少代码片段，很有启发作用。
