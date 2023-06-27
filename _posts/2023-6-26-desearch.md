# 简单解释 DeSearch

如果分布式账本也搞中心化搜索，那等于白做：内容审查、价格歧视。

如果用以太坊智能合约做搜索，那用户体验不太好：时延高且不可预测，gas fee也要花不少。

想构建一类搜索：有去中心化的自由不受控制，也有中心化搜索的时效性。

探索：去中心化的搜索引擎！

学习了FaaS，深受启发。“无状态算子+有状态存储”的存算分离思路能轻松化解节点的随时进出问题。

有状态存储显然可以是区块链。或者，用一个云存储，外加区块链，兼具前者的高效，亦具后者的安全。

无状态算子可以是冗余算力的多数投票。或者，可信算力构建的网络。

但是，离散化的计算如何提供可验证的结果呢？非要想个可验证数据结构不可，如默克尔树。Witness其实就是数据依赖的哈希树。

讨论：
- 中心化TEE固然可以满足需要，但会不会突然停止服务404？
- Kanban被banned住怎么办？只要算力还在，就能轻松恢复。Kanban也能化身为IPFS等其他公有存储。


# 后记

想要颠覆区块链：不是颠覆区块链的设计理念，而是颠覆区块链的实现方法。

让上链变快：想要易于交流的链，大家可以随时沟通，只是沟通后的东西需要过段时间才会被“持久化”。

让验证变得简单：想要易于验证的链，不需要重头来。这点是可行的，就是将验证状态的检查点持久化，“前人栽树，后人乘凉”。

# 最初的Abstract（乌托邦式）

Today's internet is dominated by certain gatekeepers, such as search engines, social media platforms, news feeds, and more. Decentralization ensures that the communication nodes are not controlled by a monopoly of minority institutes, organizations, or governments.

The emergence of a large number of decentralized applications and systems suggests an urgent need in this field. However, academia has not yet examined the trusted model in depth, especially in the context of decentralized systems. Previous approaches, such as sMPC or FHE, target applications with specific characteristics, and their ability to generalize is insufficient. Additionally, their performance overhead is relatively high, which hinders their practicality.

We surveyed existing decentralized applications and found a popular misunderstanding of security issues and inadequate security assurances. Our goal is to present a general framework that ensures the application not only meets the need for decentralization but also incorporates the ideal security features. We reproduced these scenarios with the aid of our system to meet all security requirements at a reasonable cost. Our work contributes to returning the Internet to its original purpose as a democratic, autonomous environment.
