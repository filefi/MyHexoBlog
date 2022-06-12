---
title: MPLS 实验1
date: 2014-03-14 19:00:41
updated: 2014-03-14 19:00:41
tags: [MPLS, Network]
categories: Network
---


# 实验环境：
- 模拟器：GNS3 0.8.6
- 路由器IOS：c7200-adventerprisek9-mz.151-4.M2.image
 
# GNS3实验拓扑文件：
[拓扑文件](topology.net)

# 实验拓扑：
![](topo.png)


<!-- more -->

# 基本预配置：
## R1：
```
hostname R1
!
ip cef
!
interface Loopback0
    ip address 1.1.1.1 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.12.1 255.255.255.0
    ip ospf 1 area 0
    no shut
!
interface FastEthernet0/1
    ip address 192.168.15.1 255.255.255.0
    no shut
!
router ospf 1
    router-id 1.1.1.1
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```
  
  
## R2：
```
hostname R2
!
ip cef
!
interface Loopback0
    ip address 2.2.2.2 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.12.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.23.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
router ospf 1
    router-id 2.2.2.2
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```
  
 
## R3：
```
hostname R3
!
ip cef
!
interface Loopback0
    ip address 3.3.3.3 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.34.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.23.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
router ospf 1
    router-id 3.3.3.3
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```
  
 
## R4：
```
hostname R4
!
ip cef
!
interface Loopback0
    ip address 4.4.4.4 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.34.4 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.47.4 255.255.255.0
    no shutdown
!
router ospf 1
    router-id 4.4.4.4
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```
  
  
 
# 实验与调试（本实验仅涉及AS1中的4台路由器）：
## 实验1：MPLS基本配置与验证：AS1作为MPLS域，在AS1中的所有路由器上运行MPLS；
  
可以使用命令`mpls label protocol {ldp | tdp}` ,来指定标签分发协议；
可以看到支持两种标签分发协议，LDP和TDP，默认使用LDP作为标签分发协议；
```
R1(config)#mpls label protocol ?
 ldp  Use LDP (default)
 tdp  Use TDP

R1(config)#mpls label protocol ldp
```
  
可以使用命令`mpls ldp router-id interface [force]`来强制改变LDP路由ID；
如果不指定LDP router-id则LDP router-id选举规则同 OSPF ；
```c
//手动指定R1将l0接口地址作为LDP router-id的一部分，force选项表示立即生效；
R1(config)#mpls ldp router-id l0 force
```

使用接口命令`mpls ip`，在R1所有已激活的接口上启用MPLS ：
```c
R1(config-if)#interface FastEthernet0/0
R1(config-if)#mpls ip
```

可以看到在R1的loopback 0接口配置启用MPLS时，IOS日志提示回环接口不支持MPLS ：
```c
R1(config-if)#interface Loopback0
R1(config-if)#mpls ip
% MPLS not supported on interface Loopback0
```


在R2上，使用 LDP 作为 MPLS 标签分发协议，指定 loopback 0 作为 LDP 的 `router-id`，并使用接口命令 `mpls ip` ，在适当的的接口上启用 MPLS ；
```c
R2(config)#mpls label protocol ldp
R2(config)#mpls ldp router-id l0 force

R2(config-if)#interface FastEthernet0/0
R2(config-if)#mpls ip

*Mar  1 00:09:05.223: %LDP-5-NBRCHG: LDP Neighbor 1.1.1.1:0 (1) is UP

// 可以看到LDP邻居变化日志提示，LDP邻居已经UP，邻居为1.1.1.1:0；

R2(config-if)#interface FastEthernet0/1
R2(config-if)#mpls ip
```
   
在R3上，使用LDP作为MPLS标签分发协议，指定loopback 0作为LDP的`router-id`，并使用接口命令`mpls ip`，在适当的的接口上启用MPLS；
```
R3(config)#mpls label protocol ldp
R3(config)#mpls ldp router-id l0 force

R3(config-if)#interface FastEthernet0/0
R3(config-if)#mpls ip
R3(config-if)#interface FastEthernet0/1
R3(config-if)#mpls ip
```

在R4上，使用 LDP 作为 MPLS 标签分发协议，指定 loopback 0 作为 LDP 的`router-id`，并使用接口命令`mpls ip`，在适当的的接口上启用 MPLS ；
```
R4(config)#mpls label protocol ldp
R4(config)#mpls ldp router-id l0 force

R4(config-if)#interface FastEthernet0/0
R4(config-if)#mpls ip
```

**抓包查看分析LDP包结构：**
![](pic1.png)
![](pic2.png)
可以看到LDP hello包使用UDP进行封装，而Keep Alive包使用TCP进行封装；


**附件：**[Wireshark抓包文件](LDP.pcapng)


在 R1 上查看启用 MPLS 的接口：
```
R1(config)#do show mpls interface
Interface              IP            Tunnel   BGP Static Operational
FastEthernet0/0        Yes (ldp)     No       No  No     Yes
```
R1上只用`f0/0`启用了 MPLS ，可以看到 MPLS 使用 LDP 作为标签分发协议；
  

在R1上查看LDP邻居发现：
```
R1#show mpls ldp discovery 
 Local LDP Identifier:
  1.1.1.1:0
  Discovery Sources:
  Interfaces:
    FastEthernet0/0 (ldp): xmit/recv
      LDP Id: 2.2.2.2:0
```
- 可以看到R1本地的LDP标识符为`1.1.1.1:0`，其中LDP路由器标识符由两部分组成：
  - 其中第一部分为接口IP地址；
  - 第二部分为LSR所使用的标签空间字节。如果为0则表示LSR是使用整个设备的标签空间，如果不为0则表示LSR是使用接口标签空间；
- 还可以看出，R1在启用LDP的接口f0/0上发现了LDP邻居;


在R1上查看LDP邻居发现详细信息：
```c
R1#show mpls ldp discovery detail 
    Local LDP Identifier:
    1.1.1.1:0
    Discovery Sources:
    Interfaces:
        FastEthernet0/0 (ldp): xmit/recv
            Enabled: Interface config    
            Hello interval: 5000 ms; Transport IP addr: 1.1.1.1   //Discovery hello时间间隔为5秒；
            LDP Id: 2.2.2.2:0
                Src IP addr: 192.168.12.2; Transport IP addr: 2.2.2.2
                Hold time: 15 sec; Proposed local/peer: 15/15 sec  //Discovery保持时间为15秒；
                Reachable via 2.2.2.2/32
```
  
在R1上查看LDP邻居关系：
```c
R1#show mpls ldp neighbor 
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 2.2.2.2.13634 - 1.1.1.1.646
        State: Oper; Msgs sent/rcvd: 643/640; Downstream
        Up time: 09:11:15
        LDP discovery sources:
            FastEthernet0/0, Src IP addr: 192.168.12.2
        Addresses bound to peer LDP Ident:
            192.168.12.2    192.168.23.2    2.2.2.2   
```
- 可以看到R1有一个LDP邻居，并且邻居标识符为2.2.2.2:0；
- 而R1本地的LDP标识符为1.1.1.1:0;TCP连接为2.2.2.2的13634端口到1.1.1.1的646端口；
- 邻居已经建立了9个小时11分15秒；
- LDP发现源为f0/0接口，地址为f0/0的接口地址192.168.12.2；


在R1上`traceroute 4.4.4.4`：
```c
R1(config-if)#do traceroute 4.4.4.4
Type escape sequence to abort.
Tracing the route to 4.4.4.4
VRF info: (vrf in name/id, vrf out name/id)
    1 192.168.12.2 [MPLS: Label 18 Exp 0] 76 msec 80 msec 76 msec
    2 192.168.23.3 [MPLS: Label 18 Exp 0] 96 msec 44 msec 68 msec
    3 192.168.34.4 120 msec *  116 msec
```
可以看到 MPLS 已经生效，`traceroute`的 ICMP 包当前正是使用 MPLS 进行转发；


   
## 实验2：MPLS LDP自动配置；

在R4上查看启用MPLS的接口详细信息：
```c
R4(config-if)#do show mpls interface detail
Interface FastEthernet0/0:
  IP labeling enabled (ldp): 
   Interface config
  LSP Tunnel labeling not enabled
  BGP labeling not enabled
  MPLS operational
  MTU = 1500
//可以看到接口F0/0已经启用了LDP，并且是在接口下通过手动配置启用的；

R4(config-if)#do sh mpls ldp discovery detail
 Local LDP Identifier:
 4.4.4.4:0
 Discovery Sources:
 Interfaces:
  FastEthernet0/0 (ldp): xmit/recv
   Enabled: Interface config
   Hello interval: 5000 ms; Transport IP addr: 4.4.4.4 
   LDP Id: 3.3.3.3:0
    Src IP addr: 192.168.34.3; Transport IP addr: 3.3.3.3
    Hold time: 15 sec; Proposed local/peer: 15/15 sec
    Reachable via 3.3.3.3/32
    Password: not required, none, in use
   Clients: IPv4
//通过查看R4的LDP邻居发现详情也可以看到接口F0/0是通过在接口下手动配置启用的MPLS LDP；
```

禁用之前在接口下启用的MPLS：
```
R4(config)#int f0/0
R4(config-if)#no mpls ip
```

在R4的OSPF路由器配置模式下启用MPLS LDP自动配置；
```
R4(config)#router ospf 1
R4(config-router)#mpls ldp autoconfig 
```

在R4上验证：
```c
R4(config-router)#do sh mpls interface detail
Interface FastEthernet0/0:
  IP labeling enabled (ldp):
   IGP config
  LSP Tunnel labeling not enabled
  BGP labeling not enabled
  MPLS operational
  MTU = 1500
//可以看到，接口F0/0现在已经启用了LDP，但现在是通过IGP自动配置的方式启用的；

R4(config-router)#do show mpls ldp discovery detail
 Local LDP Identifier:
 4.4.4.4:0
 Discovery Sources:
 Interfaces:
  FastEthernet0/0 (ldp): xmit/recv
   Enabled: IGP config;
   Hello interval: 5000 ms; Transport IP addr: 4.4.4.4 
   LDP Id: 3.3.3.3:0
    Src IP addr: 192.168.34.3; Transport IP addr: 3.3.3.3
    Hold time: 15 sec; Proposed local/peer: 15/15 sec
    Reachable via 3.3.3.3/32
    Password: not required, none, in use
   Clients: IPv4
R4(config-router)#do sh mpls interface detail
Interface FastEthernet0/0:
  IP labeling enabled (ldp):
   IGP config
  LSP Tunnel labeling not enabled
  BGP labeling not enabled
  MPLS operational
  MTU = 1500
//可以看到，接口F0/0现在已经启用了LDP，但现在是通过IGP自动配置的方式启用的；
```
> 注意：MPLS LDP自动配置目前只支持OSPF；

  
## 实验3：修改MPLS LDP 计时器；

可以使用命令`show mpls ldp parameters`查看 LDP 当前使用的计时器参数：
```c
R1#show mpls ldp parameters 
Protocol version: 1
Downstream label generic region: min label: 16; max label: 100000
Session hold time: 180 sec; keep alive interval: 60 sec   //LDP TCP会话保持时间为180秒，LDP TCP会话Keep Alive消息发送间隔为60秒；
Discovery hello: holdtime: 15 sec; interval: 5 sec  //LDP Discovery hello保持时间为15秒，hello消息发送时间间隔为5秒；
Discovery targeted hello: holdtime: 90 sec; interval: 10 sec  //目标邻居Discovery hello保持时间为90秒，hello消息发送间隔为10秒；
Downstream on Demand max hop count: 255
Downstream on Demand Path Vector Limit: 255
LDP for targeted sessions
LDP initial/maximum backoff: 15/120 sec
LDP loop detection: off
```

在R1上修改MPLS LDP邻居发现hello 发送间隔和保持时间：
```c
//将R1的MPLS LDP邻居发现hello时间间隔设置为10秒；
R1(config)#mpls ldp discovery hello interval 10
//将R1的MPLS LDP邻居发现保持时间设置为30秒；
R1(config)#mpls ldp discovery hello holdtime 30
```

在R1上验证MPLS LDP参数信息：
```c
R1(config)#do sh mpls ldp parameter
LDP Feature Set Manager: State Initialized
 LDP features:
 Basic
 IP-over-MPLS
 TDP
 IGP-Sync
 Auto-Configuration
 TCP-MD5-Rollover
Protocol version: 1
Session hold time: 180 sec; keep alive interval: 60 sec
Discovery hello: holdtime: 30 sec; interval: 10 sec
Discovery targeted hello: holdtime: 90 sec; interval: 10 sec
Downstream on Demand max hop count: 255
LDP for targeted sessions
LDP initial/maximum backoff: 15/120 sec
LDP loop detection: off
//可以看到R1的LDP Discovery计时器的hello interval被修改为30秒，holdtime被修改为10秒；
```

验证LDP Discovery 计时器：
```c
R1#show mpls ldp neighbor 2.2.2.2 detail
 Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
  TCP connection: 2.2.2.2.42221 - 1.1.1.1.646
  Password: not required, none, in use
  State: Oper; Msgs sent/rcvd: 82/81; Downstream; Last TIB rev sent 16
  Up time: 01:02:47; UID: 2; Peer Id 0;
  LDP discovery sources:
   FastEthernet0/0; Src IP addr: 192.168.12.2 
   holdtime: 15000 ms, hello interval: 5000 ms
  Addresses bound to peer LDP Ident:
   192.168.12.2    192.168.23.2    2.2.2.2         
  Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
  Capabilities Sent:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
  Capabilities Received:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]

R1#show mpls ldp discovery detail
 Local LDP Identifier:
 1.1.1.1:0
 Discovery Sources:
 Interfaces:
  FastEthernet0/0 (ldp): xmit/recv
   Enabled: Interface config
   Hello interval: 5000 ms; Transport IP addr: 1.1.1.1 
   LDP Id: 2.2.2.2:0
    Src IP addr: 192.168.12.2; Transport IP addr: 2.2.2.2
    Hold time: 15 sec; Proposed local/peer: 30/15 sec
    Reachable via 2.2.2.2/32
    Password: not required, none, in use
   Clients: IPv4
//可以看到邻居LSR的LDP Discovery hello interval为5秒,holdtime为15秒；
//本地的Holdtime为30秒；
```

在R1上修改LDP TCP会话保持时间设置为300秒：
```c
R1(config)#mpls ldp holdtime 300 
%Previously established sessions may not use the new holdtime.
//要让新的保持时间立即生效，需要重置一下LDP邻居关系；
R1(config)#do clear mpls ldp neighbor *
```
> 注意：LDP TCP会话保持时间被修改后，LDP会话的Keep Alive计时器值自动被修改为保持时间的1/3；

验证R1的MPLS LDP TCP会话保持时间：
```c
R1(config)#do sh mpls ldp parameters
LDP Feature Set Manager: State Initialized
 LDP features:
 Basic
 IP-over-MPLS
 TDP
 IGP-Sync
 Auto-Configuration
 TCP-MD5-Rollover
Protocol version: 1
Session hold time: 300 sec; keep alive interval: 100 sec
Discovery hello: holdtime: 30 sec; interval: 10 sec
Discovery targeted hello: holdtime: 90 sec; interval: 10 sec
Downstream on Demand max hop count: 255
LDP for targeted sessions
LDP initial/maximum backoff: 15/120 sec
LDP loop detection: off
//可以看到R1本地的LDP会话保持时间已经为300秒，其Keep Alive Interval自动被设置为100秒；
```

查看邻居LDP对等体的MPLS LDP TCP会话保持时间：
```c
R1(config)#do sh mpls ldp neighbor 2.2.2.2 detail                         
 Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
  TCP connection: 2.2.2.2.21743 - 1.1.1.1.646
  Password: not required, none, in use
  State: Oper; Msgs sent/rcvd: 15/14; Downstream; Last TIB rev sent 16
  Up time: 00:04:00; UID: 3; Peer Id 0;
  LDP discovery sources:
   FastEthernet0/0; Src IP addr: 192.168.12.2 
   holdtime: 15000 ms, hello interval: 5000 ms
  Addresses bound to peer LDP Ident:
   192.168.12.2    192.168.23.2    2.2.2.2         
  Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
  Capabilities Sent:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
  Capabilities Received:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
```
- 注意到邻居对等体的保持时间仍然为180秒，Keep Alive Interval仍然是60秒；其原因在于，和LDP Discovery计时器取较小值相同，LDP TCP会话计时器也取较小值；
- 由于刚刚修改LDP 会话保持时间为300秒，此值大于默认的180秒；


修改R2的LDP TCP会话保持时间改为900秒：
```c
R2(config)#mpls ldp holdtime 900
%Previously established sessions may not use the new holdtime.

R2(config)#do clear mpls ldp neighbor *
```

可以看到R2本地的LDP TCP会话保持时间已经被设置为900秒，KA Interval自动被设置为300秒：
```c
R2(config)#do show mpls ldp parameters
LDP Feature Set Manager: State Initialized
 LDP features:
 Basic
 IP-over-MPLS
 TDP
 IGP-Sync
 Auto-Configuration
 TCP-MD5-Rollover
Protocol version: 1
Session hold time: 900 sec; keep alive interval: 300 sec
Discovery hello: holdtime: 15 sec; interval: 5 sec
Discovery targeted hello: holdtime: 90 sec; interval: 10 sec
Downstream on Demand max hop count: 255
LDP for targeted sessions
LDP initial/maximum backoff: 15/120 sec
LDP loop detection: off
```
   
回到R1上查看LDP 邻居对等体R2实际使用的LDP会话保持时间：
```c
R1(config)#do sh mpls ldp neighbor 2.2.2.2 detail
 Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
  TCP connection: 2.2.2.2.37641 - 1.1.1.1.646
  Password: not required, none, in use
  State: Oper; Msgs sent/rcvd: 12/12; Downstream; Last TIB rev sent 16
  Up time: 00:02:58; UID: 5; Peer Id 0;
  LDP discovery sources:
   FastEthernet0/0; Src IP addr: 192.168.12.2 
   holdtime: 15000 ms, hello interval: 5000 ms
  Addresses bound to peer LDP Ident:
   192.168.12.2    192.168.23.2    2.2.2.2         
  Peer holdtime: 300000 ms; KA interval: 100000 ms; Peer state: estab
  Capabilities Sent:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
  Capabilities Received:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
```
- 可以看到虽然R2本地的LDP 会话保持时间已经被设置为900秒，但是在与R1协商后，由于R1的LDP会话保持时间300秒小于R2的900秒，所以R2使用300秒作为LDP会话保持时间；
> **注意，LDP邻居计时器可以不一致，当两台LDP邻居对等体在交换参数的时候，取较小的计时器值，Discovery计时器如此，LDP TCP会话保持时间亦如此，基于目标的LDP邻居会话同样如此；**


## 实验4：LDP认证：在R1和R2之间配置LDP身份认证

为避免等待验证实验现象时间较长，修改LDP 会话保持时间为15秒：
```c
R1(config)#mpls ldp holdtime 15
```

在R1上针对邻居2.2.2.2启用LDP认证：
```c
R1(config)#mpls ldp neighbor 2.2.2.2 password cisco
```

**注意：实验中，发现如果不重置LDP邻居对等体关系，LDP身份认证配置无法生效；**
```c
R1(config)#do clear mpls ldp nei 2.2.2.2  
R1(config)#
*Mar 13 21:36:40.426: %LDP-5-CLEAR_NBRS: Clear LDP neighbors (2.2.2.2) by console
*Mar 13 21:36:40.466: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (1) is DOWN (User cleared session manually)
R1(config)#
*Mar 13 21:36:42.822: %TCP-6-BADAUTH: No MD5 digest from 2.2.2.2(11164) to 1.1.1.1(646)
```

由于R2还未配置LDP认证，重置LDP邻居关系后，出现TCP认证报错日志，没有MD5摘要来自2.2.2.2：
```c
//在R2上针对邻居1.1.1.1启用LDP认证；
R2(config)#mpls ldp neighbor 1.1.1.1 password cisco    
//注意：实验中，发现如果不重置LDP邻居对等体关系，LDP身份认证配置无法生效；
R2(config)#do clear mpls ldp nei 1.1.1.1
```

  
## 实验5：基于目标的LDP会话

配置R1与R4建立基于目标的远程LDP会话
```c
//在R1上指定远程LDP邻居；
 R1(config)#mpls ldp neighbor 4.4.4.4 targeted
 //注意，如果一台LSR只配置命令mpls ldp neighbor ip-address targeted，则这台LSR为远程LDP会话的主动发起方；

//设置的基于目标的LDP邻居会话Discovery Hello Interval和Holdtime；
 R1(config)#mpls ldp discovery targeted-hello interval 10
 R1(config)#mpls ldp discovery targeted-hello holdtime 30

//配置R1只允许R4与自己建立远程LDP邻居，通过配置R1只接受R4发送给自己的Discovery Hello消息来实现；
 R1(config)#mpls ldp discovery targeted-hello accept from 1
 R1(config)#access-list 1 permit host 4.4.4.4
 //注意，如果一台LSR只配置命令mpls ldp discovery targeted-hello accept from，则这台LSR为远程LDP会话的被动接受方；
 //注意：当一台LSR只配置此命令，并不是说这台LSR只能接收主动发起方通告来的标签绑定信息，
 //即使，一台LSR只配置了此命令，当其与远程LDP会话主动发起方建立LDP远程会话之后，
 //被动接受方LSR也会向主动发起方LSR通告自己的入标签；
```

在R4上指定远程LDP邻居；
```c
R4(config)#mpls ldp neighbor 1.1.1.1 targeted

//配置R4只允许R1与自己建立远程LDP邻居，通过配置R4只接受R1发送给自己的Discovery Hello消息来实现；
R4(config)#mpls ldp discovery targeted-hello accept from 1
R4(config)#access-list 1 permit host 1.1.1.1

//注意：如果一台LSR同时配置了命令mpls ldp discovery targeted-hello accept from和命令mpls ldp neighbor ip-address targeted，
//则这台路由器既是远程基于目标LDP会话的主动发起方，同时也是被动接受方；
```

验证R1基于目标LDP会话计时器配置
```c
R1(config)#do sh mpls ldp parameters               
LDP Feature Set Manager: State Initialized
 LDP features:
 Basic
 IP-over-MPLS
 TDP
 IGP-Sync
 Auto-Configuration
 TCP-MD5-Rollover
Protocol version: 1
Session hold time: 15 sec; keep alive interval: 5 sec
Discovery hello: holdtime: 30 sec; interval: 10 sec
Discovery targeted hello: holdtime: 30 sec; interval: 10 sec
Accepting targeted hellos; peer acl: 1
Downstream on Demand max hop count: 255
LDP for targeted sessions
LDP initial/maximum backoff: 15/120 sec
LDP loop detection: off
//可以看到R1本地基于目标的LDP会话Discovery Hello Interval为10秒，Holdtime为30秒；
//R1接受基于目标的Hello消息，并且基于ACL1来进行远程邻居对等体过滤；
```

在R1上查看MPLS LDP邻居R4的信息；
```c
R1(config)#do show mpls ldp neighbor 4.4.4.4 detail
 Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
  TCP connection: 4.4.4.4.61631 - 1.1.1.1.646
  Password: not required, none, in use
  State: Oper; Msgs sent/rcvd: 75/75; Downstream; Last TIB rev sent 17
  Up time: 00:04:44; UID: 8; Peer Id 2;
  LDP discovery sources:
   Targeted Hello 1.1.1.1 -> 4.4.4.4, active, passive;//R1与R4已建立远程LDP会话；关键字active和passive表示R1是远程会话的主动发起方和被动接受方；
   holdtime: infinite, hello interval: 3333 ms
  Addresses bound to peer LDP Ident:
   192.168.34.4    192.168.47.4    4.4.4.4         
  Peer holdtime: 15000 ms; KA interval: 5000 ms; Peer state: estab
  Clients: Dir Adj Client
  Capabilities Sent:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
  Capabilities Received:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
```

在R4上查看MPLS LDP邻居R1的信息；
```c
R4(config)#do show mpls ldp neighbor 1.1.1.1 detail
 Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
  TCP connection: 1.1.1.1.646 - 4.4.4.4.61631
  Password: not required, none, in use
  State: Oper; Msgs sent/rcvd: 86/85; Downstream; Last TIB rev sent 17
  Up time: 00:05:31; UID: 5; Peer Id 2;
  LDP discovery sources:
   Targeted Hello 4.4.4.4 -> 1.1.1.1, active, passive;//R1与R4已建立远程LDP会话；关键字active和passive表示R1是远程会话的主动发起方和被动接受方；
   holdtime: infinite, hello interval: 3333 ms
  Addresses bound to peer LDP Ident:
   192.168.12.1    192.168.15.1    1.1.1.1         
  Peer holdtime: 15000 ms; KA interval: 5000 ms; Peer state: estab
  Clients: Dir Adj Client
  Capabilities Sent:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
  Capabilities Received:
   [ICCP (type 0x0405) MajVer 1 MinVer 0]
   [Dynamic Announcement (0x0506)]
   [Typed Wildcard (0x050B)]
```


## 实验6：控制LDP标签通告；

配置MPLS LDP来向特定的LDP对等体通告或者不通告特定的标签；
```c
//在R4上查看LIB;
R4#show mpls ldp bindings
 lib entry: 1.1.1.1/32, rev 14
  local binding:  label: 19
  remote binding: lsr: 3.3.3.3:0, label: 18
  remote binding: lsr: 1.1.1.1:0, label: imp-null
 lib entry: 2.2.2.2/32, rev 12
  local binding:  label: 18
  remote binding: lsr: 3.3.3.3:0, label: 17
  remote binding: lsr: 1.1.1.1:0, label: 16
 lib entry: 3.3.3.3/32, rev 8
  local binding:  label: 16
  remote binding: lsr: 3.3.3.3:0, label: imp-null
  remote binding: lsr: 1.1.1.1:0, label: 19
 lib entry: 4.4.4.4/32, rev 2
  local binding:  label: imp-null
  remote binding: lsr: 3.3.3.3:0, label: 16
  remote binding: lsr: 1.1.1.1:0, label: 18
 lib entry: 192.168.12.0/24, rev 16
  local binding:  label: 20
  remote binding: lsr: 3.3.3.3:0, label: 19
  remote binding: lsr: 1.1.1.1:0, label: imp-null
 lib entry: 192.168.15.0/24, rev 17
  remote binding: lsr: 1.1.1.1:0, label: imp-null
 lib entry: 192.168.23.0/24, rev 10
  local binding:  label: 17
  remote binding: lsr: 3.3.3.3:0, label: imp-null
  remote binding: lsr: 1.1.1.1:0, label: 17
 lib entry: 192.168.34.0/24, rev 6
  local binding:  label: imp-null
  remote binding: lsr: 3.3.3.3:0, label: imp-null
  remote binding: lsr: 1.1.1.1:0, label: 20
 lib entry: 192.168.47.0/24, rev 4
  local binding:  label: imp-null

R4#show mpls ip binding
 1.1.1.1/32 
  in label:     19        
  out label:    18        lsr: 3.3.3.3:0        inuse
  out label:    imp-null  lsr: 1.1.1.1:0       
 2.2.2.2/32 
  in label:     18        
  out label:    17        lsr: 3.3.3.3:0        inuse
  out label:    16        lsr: 1.1.1.1:0       
 3.3.3.3/32 
  in label:     16        
  out label:    imp-null  lsr: 3.3.3.3:0        inuse
  out label:    19        lsr: 1.1.1.1:0       
 4.4.4.4/32 
  in label:     imp-null  
  out label:    16        lsr: 3.3.3.3:0       
  out label:    18        lsr: 1.1.1.1:0       
 192.168.12.0/24 
  in label:     20        
  out label:    19        lsr: 3.3.3.3:0        inuse
  out label:    imp-null  lsr: 1.1.1.1:0       
 192.168.15.0/24 
  out label:    imp-null  lsr: 1.1.1.1:0       
 192.168.23.0/24 
  in label:     17        
  out label:    imp-null  lsr: 3.3.3.3:0        inuse
  out label:    17        lsr: 1.1.1.1:0       
 192.168.34.0/24 
  in label:     imp-null  
  out label:    imp-null  lsr: 3.3.3.3:0       
  out label:    20        lsr: 1.1.1.1:0       
 192.168.47.0/24 
  in label:     imp-null
 ```


**在R3上进行配置，让R3只将与环回口的前缀相绑定的标签通告给R4；**
```c
//配置ACL以确定准许通告的特定前缀；
R3(config)#ip access-list standard PERMIT_LOOPBACK 
R3(config-std-nacl)#permit
R3(config-std-nacl)#permit host 1.1.1.1
R3(config-std-nacl)#permit host 2.2.2.2
R3(config-std-nacl)#permit host 3.3.3.3 
R3(config-std-nacl)#exit

//配置ACL以确定特定的LDP对等体可以接收标签绑定信息；
R3(config)#ip access-list standard PEER_R4
R3(config-std-nacl)#permit host 4.4.4.4 
R3(config-std-nacl)#exit

//配置LDP不向外通告任何标签；
R3(config)#no mpls ldp advertise-labels

//配置LDP向对等体PEER_R4通告前缀PERMIT_LOOPBACK的标签绑定信息；
R3(config)#mpls ldp advertise-labels for PERMIT_LOOPBACK to PEER_R4

//查看R3的标签通告信息；
R3(config)#do show mpls ldp bindings advertisement-acls
Advertisement spec:
 Prefix acl = PERMIT_LOOPBACK; Peer acl = PEER_R4

lib entry: 1.1.1.1/32, rev 22
 Advert acl(s): Prefix acl PERMIT_LOOPBACK; Peer acl PEER_R4
lib entry: 2.2.2.2/32, rev 23
 Advert acl(s): Prefix acl PERMIT_LOOPBACK; Peer acl PEER_R4
lib entry: 3.3.3.3/32, rev 24
 Advert acl(s): Prefix acl PERMIT_LOOPBACK; Peer acl PEER_R4
lib entry: 4.4.4.4/32, rev 25
lib entry: 192.168.12.0/24, rev 26
lib entry: 192.168.23.0/24, rev 27
lib entry: 192.168.34.0/24, rev 28
lib entry: 192.168.47.0/24, rev 29
//可以看到R3基于ACL PERMIT_LOOPBACK来确定需要通告的与特定前缀相绑定的标签和可以接收标签绑定信息的LDP对等体PEER_R4；
```

在R4上查看来自LDP对等体的标签绑定信息；
```c
R4#show mpls ldp bindings neighbor 3.3.3.3 detail
 lib entry: 1.1.1.1/32, rev 14, chkpt: none
  remote binding: lsr: 3.3.3.3:0, label: 18
 lib entry: 2.2.2.2/32, rev 12, chkpt: none
  remote binding: lsr: 3.3.3.3:0, label: 17
 lib entry: 3.3.3.3/32, rev 8, chkpt: none
  remote binding: lsr: 3.3.3.3:0, label: imp-null
 lib entry: 4.4.4.4/32, rev 2
 lib entry: 192.168.12.0/24, rev 16
 lib entry: 192.168.15.0/24, rev 17
 lib entry: 192.168.23.0/24, rev 10
 lib entry: 192.168.34.0/24, rev 6
 lib entry: 192.168.47.0/24, rev 4
//可以看到R4确实只收到了与环回口前缀相绑定的标签；

//查看R4的LIB；
R4#show mpls ip binding
 1.1.1.1/32 
  in label:     19        
  out label:    18        lsr: 3.3.3.3:0        inuse  //inuse表示该标签已被放入LFIB表，并用来转发流量；
  out label:    imp-null  lsr: 1.1.1.1:0       
 2.2.2.2/32 
  in label:     18        
  out label:    17        lsr: 3.3.3.3:0        inuse
  out label:    16        lsr: 1.1.1.1:0       
 3.3.3.3/32 
  in label:     16        
  out label:    imp-null  lsr: 3.3.3.3:0        inuse
  out label:    19        lsr: 1.1.1.1:0       
 4.4.4.4/32 
  in label:     imp-null  
  out label:    18        lsr: 1.1.1.1:0       
 192.168.12.0/24 
  in label:     20        
  out label:    imp-null  lsr: 1.1.1.1:0       
 192.168.15.0/24 
  out label:    imp-null  lsr: 1.1.1.1:0       
 192.168.23.0/24 
  in label:     17        
  out label:    17        lsr: 1.1.1.1:0       
 192.168.34.0/24 
  in label:     imp-null  
  out label:    20        lsr: 1.1.1.1:0       
 192.168.47.0/24 
  in label:     imp-null
//可以看到虽然R3只向R4通告了环回口前缀的标签绑定信息，
//但是由于R4和R1有远程目标对等体关系，所R4还是从R1哪里收到其他前缀的标签；

//查看R4的LFIB；
R4#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  3.3.3.3/32       0             Fa0/0      192.168.34.3
17         No Label   192.168.23.0/24  0             Fa0/0      192.168.34.3
18         17         2.2.2.2/32       0             Fa0/0      192.168.34.3
19         18         1.1.1.1/32       0             Fa0/0      192.168.34.3
20         No Label   192.168.12.0/24  0             Fa0/0      192.168.34.3
```
  
**配置MPLS LDP，使其有选择地过滤接收到的特定前缀标签绑定信息**

- 配置R4仅仅接收R1环回口1.1.1.1/32的标签绑定信息；

```c
//配置ACL以确定准许接受的特定前缀；
R4(config)#ip access-list standard PERMIT_R1_LOOPBACK
R4(config-std-nacl)#permit host 1.1.1.1
R4(config-std-nacl)#exit
 
//针对LDP邻居R1和R3应用ACL以过滤接收到的特定前缀的绑定信息；
R4(config)#mpls ldp neighbor 3.3.3.3 labels accept PERMIT_R1_LOOPBACK
R4(config)#mpls ldp neighbor 1.1.1.1 labels accept PERMIT_R1_LOOPBACK
  
//查看R4的LIB表，以验证实验配置；
R4(config)#do show mpls ldp bindings
 lib entry: 1.1.1.1/32, rev 14
  local binding:  label: 19
  remote binding: lsr: 3.3.3.3:0, label: 18
  remote binding: lsr: 1.1.1.1:0, label: imp-null
 lib entry: 2.2.2.2/32, rev 12
  local binding:  label: 18
 lib entry: 3.3.3.3/32, rev 8
  local binding:  label: 16
 lib entry: 4.4.4.4/32, rev 2
  local binding:  label: imp-null
 lib entry: 192.168.12.0/24, rev 16
  local binding:  label: 20
 lib entry: 192.168.23.0/24, rev 10
  local binding:  label: 17
 lib entry: 192.168.34.0/24, rev 6
  local binding:  label: imp-null
 lib entry: 192.168.47.0/24, rev 4
  local binding:  label: imp-null

R4(config)#do show mpls ip binding
 1.1.1.1/32 
  in label:     19        
  out label:    18        lsr: 3.3.3.3:0        inuse
  out label:    imp-null  lsr: 1.1.1.1:0       
 2.2.2.2/32 
  in label:     18        
 3.3.3.3/32 
  in label:     16        
 4.4.4.4/32 
  in label:     imp-null  
 192.168.12.0/24 
  in label:     20        
 192.168.23.0/24 
  in label:     17        
 192.168.34.0/24 
  in label:     imp-null  
 192.168.47.0/24 
  in label:     imp-null
//可以看到R4的LIB表中确实只有R1前缀1.1.1.1/32的标签绑定信息；
```
