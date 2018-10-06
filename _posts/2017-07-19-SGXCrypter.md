**题目： SGXCrypter: IP protection for portable executables using Intel's SGX technology**

**作者： Dimitrios Tychalas, Nektarios Georgios Tsoutsos, Michail Maniatakos**

**单位： New York University Abu Dhab & NYU Tandon School of Engineering**

**出版： ASP-DAC 2017**

#解决问题：
1. 处于IP保护，常见的二进制代码混淆技术有压缩和加密，但二进制镜像内同样附有解压缩和解密的代码，可作为逆向攻击的突破口；
2. 现代加壳技术会留有一部分unpacking stub，用于引导到真正的入口点，这个stub并不加密，是静态分析的众矢之的；
3. 任何数据混淆技术本质都是静态的，在运行时分析便失去隐藏的能力。
4. 本工作借助硬件技术，解决了Windows系统下PE二进制文件的IP问题，能抵抗静态和动态分析。


#贡献点：
1. 第一个借助SGX实现Windows PE的IP版权；
2. 借助验证attestation实现访问控制，同时保护密钥；
3. 和流行的反病毒引擎兼容，方便无缝部署。


#相关工作：
* Hyperion：也是PE crypter，能有效抵抗静态分析攻击，但是性能开销巨大。
* White Box Cryptography：加密密钥的随机化布局，动静都可造成一定阻碍，缺点：慢和臃肿。


#设计：
和传统的加壳技术基本一致：
    1. Payload Encryption
    2. Decryption Stub：can be executed on any SGX-enabled machine
    3. Stub Execution
    4. Remote Authentication：使出了_必杀绝技_，但文章这里没有考虑重放攻击
    5. Payload Execution

* 防御静态分析：密钥由第三方提供，在远程验证时获取。使用AES-128加密信道，用增强版Diffie-Hellman密钥交换协议 *SIGMA* 。
* 防御动态分析：完全借助SGX，内存地址不可访问(EPC)，内容加密(MEE)，不可调试等。


#实验：
1. 能成功逃逸35种反病毒引擎的检测；
2. 加载时间平均每MB慢了0.6秒。


#局限性：
1. 对Linux的二进制文件支持留作未来工作；
2. SGX对可执行文件不支持Windows API，而且利用SGX的程序需要额外设计，对二进制而言需要重编译，工作量大（言下之意，本工作的实用性还不强）；
3. EPC内存有限，所有加固的程序都会引入昂贵的换页开销（SGX自身局限）。


#体会：
1. 结合的想法很好。可能时间仓促，设计比较简单，实验部分虽然有效但还可加入更多具有说服力的论据。
2. 属于novel idea文章。
