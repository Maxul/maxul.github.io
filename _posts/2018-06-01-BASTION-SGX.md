**题目： BASTION-SGX: bluetooth and architectural support for trusted I/O on SGX**

**作者： Travis Peters, Reshma Lal, Srikanth Varadarajan, Pradeep Pappachan, David Kotz**

**单位： Dartmouth College and Intel Corporation**

**出版： HASP@ISCA 2018**

# 解决问题：

1.    对某些应用程序而言（如密码），其IO数据和在TEE内部保护的数据一样具备敏感性。
2.    传统的思路引入了可信路径，通常要引入可信驱动、中间件、系统或VM/HV。这个做法对SGX程序而言不方便的是Enclave很难对Enclave的东西做attest。
3.    可信路径问题：banking app, health app, messaging app都需要一条和特定外设连接的安全通路。malware可以破坏上述探讨任何一种软件来破坏之前设定的可信路径。
4.    本文关注无线通信中的蓝牙，针对蓝牙设备和SGX程序建立可信路径，并在未来推广到WiFI和NFC等短距离通信。


# 贡献：

1.    指出并解决针对蓝牙设备的可信IO挑战；
2.    给出BASTION-SGX架构，对蓝牙固件进行扩展，实现可信IO功能。
3.    PoC使用的HID的键盘鼠标作为例子。


# 威胁模型：

*    只关注软件攻击：非法数据访问，数据注入和重放攻击。


# 方案：
* 引入可信硬件固件，修改其协议层，即对蓝牙设备添加特定协议，使之可以与SGX-Platform上的Trusted Apps建立安全信道，保证传输数据的保密性、完整性。详见原文图4。包协议见图6。

    1. Packet Multiplexing & Interleaving
    2. Isolating & Securing User Data Only


# 体会：

1.    关于SGX的可信IO工作已将拉开厮杀序幕。
2.    本文引入更多可信硬件（蓝牙）和固件（HCI）来实现可信路径，本质是需要一个特定协议的硬件，而非一个针对任意外设的框架。
3.    SGX不仅用于云端，也用于IoT。
