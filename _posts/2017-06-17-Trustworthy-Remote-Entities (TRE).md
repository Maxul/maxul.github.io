**题目： Exploring the use of Intel SGX for Secure Many-Party Applications**

**作者： Kubilay Ahmet Küçük, Andrew Paverd, Andrew C. Martin, N. Asokan, Andrew Simpson, Robin Ankele**

**单位： University of Oxford, United Kingdom**

**出版： SysTEX@Middleware 2016**

## 名词解释： 
* Trusted Third Party (TTP)：所有软件组件都信任的实体。 

## 解决的问题： 
* 在现代软件发展中，我们大量使用第三方的库或程序来集成完善我们的功能。在代码复用上是件好事，但在安全上却缺乏合理有效的解决方案来处理彼此间互不信任的问题。TTP本质上是出现一个大家都一致认同的信任机构，把自己的数据委托给它，它去借用别的计算能力帮助你完成运算。TPP必须保证互不相信的任意两方无法窥视对方的任何数据。对于一个涉及大量第三方应用交互的程序来说，TPP会在性能上显著胜出。 
* 在LBS服务中，定位系统可能要给线上餐饮系统发送用户所在地理位置，这会导致用户数据隐私被泄露。 

## 挑战： 
* 很多分布式系统最头痛的问题就是无法保证远程访问的服务是不是依旧可信，通常这块工作的保证是借助**加密信道传输+身份认证**来实现的。
* 远程模块要提供的可信模块属于平台 root of trust 级别，通常由硬件来保证。 

## 假设： 
1. 攻击者对TRE平台拥有绝对控制权，甚至可以修改bootloader。
2. 攻击者可以任意添加或移除硬件设备，重置平台，使用旁路攻击。
3. 攻击者控制一切信道，具备Dolev–Yao模型的一切能力。
4. TRE输出的正确性只和输入相关。
（一言以蔽之，TRE的使用者可以就是坏人，但是这个平台依然可以相信。） 

## 贡献点： 
1. 对使用SGX作为TRE的应用做了详尽的性能实验分析。
2. 基于SGX的架构上原型设计与实现，为未来工作提供模板。
3. 对SGX-TRE和TPM-TRE进行了系统比较。
（这些贡献点的说明都比较苍白，创新性不是很足。实现了一个横向平台，并进行对比） 

## 参考资料： 
[牛津大学 Andrew Paverd 团队](https://www.cs.ox.ac.uk/people/andrew.paverd/tre/)：主要关注智能电网应用领域。本篇文章是该团队的最新工作成果。
