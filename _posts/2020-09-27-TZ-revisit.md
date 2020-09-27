## ARM TrustZone漏洞回顾

### SoK: Understanding the Prevailing Security Vulnerabilities in TrustZone-assisted TEE Systems | S&P2020

ARM在工业控制和智能手机上占据主导地位，其TEE扩展——TrustZone也得到了大规模应用。

本文调研了Qualcomm、Trustonic、Huawei、NVidia、Linaro等5个厂商从2013年到2018年的商业TA（运行在TZ中的程序），对其漏洞进行整理。

主要的经验教训有：
- 系统设计层面：
1. TEE内核的TCB太大：如指纹识别、GPU图像渲染等驱动最好放在用户态，使用微内核来降低TCB。
2. TEE内核暴露给TA的系统调用接口太多，导致攻击面变多：应有针对性地开放部分接口给TA，建立一套白名单机制。
3. 系统内核服务太多：应该尽可能采取微内核方式，将更多服务放到用户态，并限制和内核交互的系统调用方式。
4. 系统调用的能力的限制和检查：高通为了让TA能快速访问CA的内存，提供了映射REE内存的接口。该接口一旦被滥用，可以获取安卓root。因此系统调用应限制TA的能力——只与相应CA通信，即限制TEE内核的系统调用的空间能力。

- 系统实现层面：
5. TEE自身的调试能力在实际部署时一定要关闭，包括打印的信息等。
6. 软件级防御不到位：ASLR、Stack Canaries、Guard Pages、Execute Never等应该都开启。
7. 远程验证的第一个环节必须由硬件来保证，即硬件认为secure boot的第一步是对的，并提供相关证明。
8. TA必须有方法进行撤销或反降级处理：防止版本回滚攻击。
9. TZ的软件栈过于庞大，包括boot loader、secure monitor、Trusted kernel和TA，这里的bugs都可能导致非常严重的问题。
10. 安全机制的不当配置：外设隔离、内存保护、密钥管理都会导致严重后果。

- 硬件层面：
11. 对共享资源的并发访问，如RPMB的共享使用，可导致TOCTOU漏洞，软件自身不用constant-time实现也会导致信息泄漏。
12. SoC上有大量外设和协处理器，如FPGA，就可以通过总线进行攻击
13. CPU的电源状态也会导致某些问题产生，如REE通过电源配置攻击TEE
14. 微架构物理资源共享：cache、branch predictor、DRAM Rowhammer等问题。
