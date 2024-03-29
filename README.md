# tum-host-from-china [从中国连接到国外学校(TUM)教育网的解决方案]
This repo provides a secure and stable solution for connecting a host inside the TUM education network from CHINA 

### 注意本文 不是科学上网教程! 不是科学上网教程! 不是科学上网教程! 也不提供任何科学上网相关工具及教程. 本文着眼于:在已经可以上外网的情况下, 如何安全稳定地连接国外学校的教育网内网,来使用学校的教育网资源 (如服务器, 图书馆等), 亦可实现进行双向网络层通讯 (例如: 国外学校教育局域网机器(或IOT设备)之间的通信, 内网P2P会议等)



## 一. 问题描述
留学生上网课会遇到学校资源需要链接到学校内网才能获得, 而学校往往提供了anyconnect三层(网络层)的VPN软件来保证安全, 由于目前普遍使用的科学上网软件为四层(传输层Proxy), 所以VPN流量并不会自动走Proxy, 导致几分钟就要重新连接一次anyconnect.

## 二. 架构简述
透明路由TProxy  
自建TLP (加密)  
自建OpenVPN over Socks5 Tunnel  
以下为我大概画的一个网络结构示意图: 最终目的是图中两个红色方框内的机器可以实现网络层(IP层)相互通讯  
![structure](https://github.com/sstshenshutao/tum-host-from-china/blob/master/structure.png)

## 三. 问题
### 1. 为什么不自己搭科学上网服务, 而要去租大型的?
因为自己搭费事费力, 慢, 有政策风险. 大型的服务提供商能够给到非常好的价格, 同时保证稳定性, 而且有一些 IEPL/IPLC 线路更是普通人没法采买到的. (本文不涉及这部分, 因为有政策风险)

### 2. 为什么不直接用租用的网络(图中的SSP network), 在SSP Network层上建立OPENV? 反而要在它上面再实现一层标准的TLP(transport layer proxy)?
因为租到的网络的最后一跳不在自己的控制范围内, 而且很多服务提供商把最后一跳的很多IP层功能做了一些限制, 这些都会导致OpenVPN over SSP 不稳定, 丢包严重, 或者断连. 而自己的最后一跳可以保证一切在自己的控制范围内, 而且对于服务器有IP登录限制的, 可以用最后一跳来固定自己的IP地址. 

### 3. 为什么不直接用L4 Proxy?
VPN处于网络结构的3层, 而TLP(transport layer proxy)处于网络结构的4层, 即使透明代理TProxy也是属于4层的. 4层无法代理3层流量, 需要用到隧道技术, 也就是本文所讲的技术

### 4. 为什么要用TProxy? 
因为双重 L4-Proxy (Proxy over Proxy Tunnel)使用透明模式很容易实现, 而其他模式比如 proxy-chain等, 实现起来复杂, 需要用到客户端半边加密(算一个proxy,运行在客户端), 服务端半边解密的技术(算一个proxy,运行在服务端), 所以需要拆分修改源代码.

