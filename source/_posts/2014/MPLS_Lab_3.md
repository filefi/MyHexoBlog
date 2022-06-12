---
title: MPLS 实验3：MPLS LDP-IGP同步
date: 2014-03-14 22:28:41
updated: 2014-03-14 22:28:41
tags: [MPLS, Network]
categories: Network
---


<!-- more -->


# 实验环境：
 - 模拟器：GNS3 0.8.6
 - Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image


# GNS3实验拓扑文件：
[](topology.net)


# 实验拓扑：
![](topo.png)


# 基本预配置：
## R1：
```
hostname R1
!
ip cef
!
mpls label protocol ldp
!
interface Loopback0
    ip address 1.1.1.1 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.12.1 255.255.255.0
    ip ospf 1 area 0
    no shutdown
    mpls ip
!
interface FastEthernet0/1
    ip address 192.168.21.1 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
router ospf 1
    router-id 1.1.1.1
!
mpls ldp router-id Loopback0 force
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R2:
```
hostname R2
!
ip cef
!
mpls label protocol ldp
!
interface Loopback0
    ip address 2.2.2.2 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.12.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.21.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
router ospf 1
    router-id 2.2.2.2
!
!
mpls ldp router-id Loopback0 force
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```


# 实验与调试：
## 实验1：R1和R2之间仅有唯一一条链路时的LDP-IGP同步

将R1和R2之间的由F0/1相连的链路禁用，只留下F0/0的链路；
```c
            R1(config)#int f0/1
            R1(config-if)#shutdown
            
            R2(config)#int f0/1
            R2(config-if)#shutdown
```

查看R1的LDP邻居表；
```c
R1(config-if)#do show mpls ldp nei
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 2.2.2.2.52561 - 1.1.1.1.646
        State: Oper; Msgs sent/rcvd: 7/7; Downstream
        Up time: 00:00:11
        LDP discovery sources:
            FastEthernet0/0, Src IP addr: 192.168.12.2
        Addresses bound to peer LDP Ident:
            2.2.2.2         192.168.12.2
//可以看到，此时R1只有一个LDP邻居，即R2，并且他们之间只有一条链路，因为R1只在F0/0发现了LDP邻居；
```

在R1和R2上启用LDP-IGP同步功能；
```c
R1(config)#router ospf 1
R1(config-router)#mpls ldp sync

R2(config)#router ospf 1
R2(config-router)#mpls ldp sync
```

查看R1上F0/0的LDP和OSPF同步信息；
```c
R1(config)#do show ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required //接口f0/0的LDP-IGP同步状态为Required，说明LDP-IGP已经开启；
    Holddown timer is not configured //LDP-IGP保持计时器没有配置；
    Interface is up //接口为UP状态；
```
            
## 调试1：
清楚MPLS LDP的邻接关系；
```c
R1(config)#do clear mpls ldp neighbor *
//注意：这两条命令切换要快，否则看不到现象；
R1(config)#do show ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is not configured
    Interface is up and sending maximum metric //接口已经UP，但由于LDP邻居被重置，所以OSPF将度量值用最大值来发送给邻居；
R1(config)#
*Mar 16 09:33:16.371: %LDP-5-CLEAR_NBRS: Clear LDP neighbors (*) by console
*Mar 16 09:33:16.387: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (1) is DOWN (User cleared session manually)
R1(config)#
*Mar 16 09:33:20.507: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (1) is UP
//LDP邻居又再度建立；
```

调试MPLS LDP-IGP同步过程；

配置ACL 100以捕获R1发往R2的OSPF LSA；
```c
R1(config)#R1(config)#access-list 100 permit ospf host 1.1.1.1 any
```
**同时，开启抓包工具Wireshark，捕获F0/0上的OSPF数据包；**

调试`debug mpls ldp igp sync` 和 `debug ip ospf lsa-generation 100`；
```c
R1#debug ip ospf lsa-generation 100
OSPF LSA generation debugging is on for access list 100
R1#debug mpls ldp igp sync
LDP-IGP Synchronization debugging is on
R1#clear mpls ldp neighbor *

R1#
*Mar 16 10:51:55.823: %LDP-5-CLEAR_NBRS: Clear LDP neighbors (*) by console
*Mar 16 10:51:55.831: LDP-SYNC: Fa0/0, OSPF 1: notify status (required, not achieved, delay, holddown infinite) internal status (not achieved, timer not running)
//LDP-IGP未同步；
*Mar 16 10:51:55.835: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 10:51:55.835: LDP-SYNC: Fa0/0, 2.2.2.2: Adj being deleted, sync_achieved goes down
//LDP邻接关系被删除，同步完成状态变为down；
*Mar 16 10:51:55.843: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (1) is DOWN (User cleared session manually)
//LDP邻居关系Down；用户手动清除了会话；
*Mar 16 10:51:56.335: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x80000059
```

**在调试输出过程中飞快按下此命令，得到LDP-IGP同步过程中的OSPF Database中的一类LSA信息；**
```c
R1#sh ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

    LS age: 2
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 1.1.1.1
    Advertising Router: 1.1.1.1
    LS Seq Number: 80000059
    Checksum: 0x814F
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 1.1.1.1
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.1
        Number of MTID metrics: 0
        TOS 0 Metrics: 65535 //注意LDP-IGP同步过程中，度量值变为了最大值65535；


    LS age: 3
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 2.2.2.2
    Advertising Router: 2.2.2.2
    LS Seq Number: 8000004C
    Checksum: 0x8B45
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 2.2.2.2
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.2
        Number of MTID metrics: 0
        TOS 0 Metrics: 65535//注意LDP-IGP同步过程中，度量值变为了最大值65535；
        
R1#
*Mar 16 10:51:59.543: LDP-SYNC: Fa0/0: No session or session has not send initial update, ignore adj joining event.
*Mar 16 10:51:59.547: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (2) is UP
R1#
*Mar 16 10:51:59.551: LDP-SYNC: Fa0/0: session 2.2.2.2:0 came up, sync_achieved up
//LDP-IGP同步状态UP；
*Mar 16 10:51:59.555: LDP-SYNC: Fa0/0, OSPF 1: notify status (required, achieved, no delay, holddown infinite) internal status (achieved, timer not running)
//LDP-IGP同步完成；
*Mar 16 10:51:59.555: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 10:52:00.055: OSPF-1 LSGEN: Rate limit LSA generation for 1.1.1.1 1.1.1.1 1
R1#
*Mar 16 10:52:01.335: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x8000005A
```

```c
R1#sh ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

    LS age: 649
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 1.1.1.1
    Advertising Router: 1.1.1.1
    LS Seq Number: 8000005A
    Checksum: 0x9D31
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 1.1.1.1
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.1
        Number of MTID metrics: 0
        TOS 0 Metrics: 1//注意LDP-IGP同步完成后，度量值恢复到正常的1；


    LS age: 650
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 2.2.2.2
    Advertising Router: 2.2.2.2
    LS Seq Number: 8000004D
    Checksum: 0xA727
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 2.2.2.2
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.2
        Number of MTID metrics: 0
        TOS 0 Metrics: 1
//注意到，在OSPF邻居已经建立的情况下，OSPF并没有如预期的那样，因为LDP邻居关系的中断破裂（Down），而导致OSPF邻接关系的终端破裂（Down）；
```
                    
查看Wireshark在LDP-IGP收敛过程中捕获的数据包；
可以看到，LSA中的Metric和OSPF Database中一致；
![](pic1.png)
![](pic2.png)
                
                
继续调试`clear ip ospf process`：
```c
R1#clear ip ospf process 
Reset ALL OSPF processes? [no]: yes
R1#
*Mar 16 11:21:09.559: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.559: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Mar 16 11:21:09.563: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.563: OSPF-1 LSGEN: Scheduling network LSA on FastEthernet0/0
*Mar 16 11:21:09.567: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.567: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.571: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.575: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.575: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.603: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:09.907: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from LOADING to FULL, Loading Done
R1#
*Mar 16 11:21:09.911: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:10.059: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x80000001
R1#
*Mar 16 11:21:14.591: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:21:14.707: OSPF-1 LSMAX: Rcv Maxage LSA, Type 1, LSID 1.1.1.1, Adv rtr 1.1.1.1, age 3600, seq 0x8000005A, from 2.2.2.2, FastEthernet0/0
*Mar 16 11:21:14.707: OSPF-1 LSGEN: Update router LSA 1.1.1.1 1.1.1.1 1 8000005A
*Mar 16 11:21:14.711: OSPF-1 LSGEN: Rate limit LSA generation for 1.1.1.1 1.1.1.1 1
*Mar 16 11:21:15.059: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x8000005B
*Mar 16 11:21:15.067: OSPF-1 LSMAX: Rcv Maxage LSA, Type 1, LSID 1.1.1.1, Adv rtr 1.1.1.1, age 3600, seq 0x8000005A, from 2.2.2.2, FastEthernet0/0
R1#
*Mar 16 11:21:15.087: OSPF-1 LSGEN: Rate limit LSA generation for 1.1.1.1 1.1.1.1 1
R1#
*Mar 16 11:21:20.059: OSPF-1 LSGEN: No change in router LSA, area 0
//注意到，由于OSPF收敛迅速，所以LDP没有受到影响，甚至没有出现任何LDP日志信息；
```
                
## 调试2：

禁用R2上F0/0的MPLS，让R1和R2的LDP邻居关系断开，确保LDP会话晚于OSPF邻接关系建立；
```c
R2(config-if)#int f0/0  
R2(config-if)#no mpls ip
```

在R1上查看OSPF邻接关系；
```c
R1#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:36    192.168.12.2    FastEthernet0/0
//注意：可以看到，正如调试1中验证的一样，在OSPF邻接关系已经建立的情况下，终止LDP的邻接关系并不会影响OSPF的邻接关系；
```

此时，在经过将R2的F0/0禁用MPLS之后，R1和R2的LDP邻居已经关系；
接下来，重置OSPF进程；
```c
R1#debug ip ospf adj
OSPF adjacency debugging is on
R1#clear ip ospf process
Reset ALL OSPF processes? [no]: yes
R1#
*Mar 16 11:48:16.419: OSPF-1 ADJ   Lo0: Interface going Down
*Mar 16 11:48:16.419: OSPF-1 ADJ   Lo0: 1.1.1.1 address 1.1.1.1 is dead, state DOWN
*Mar 16 11:48:16.423: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.423: OSPF-1 ADJ   Fa0/0: Interface going Down
*Mar 16 11:48:16.423: OSPF-1 ADJ   Fa0/0: 2.2.2.2 address 192.168.12.2 is dead, state DOWN
*Mar 16 11:48:16.427: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Mar 16 11:48:16.427: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.431: OSPF-1 ADJ   Fa0/0: Neighbor change event
*Mar 16 11:48:16.431: OSPF-1 ADJ   Fa0/0: DR/BDR election
*Mar 16 11:48:16.431: OSPF-1 ADJ   Fa0/0: Elect BDR 1.1.1.1
*Mar 16 11:48:16.435: OSPF-1 ADJ   Fa0/0: Elect DR 1.1.1.1
*Mar 16 11:48:16.435: OSPF-1 ADJ   Fa0/0: Elect BDR 0.0.0.0
*Mar 16 11:48:16.435: OSPF-1 ADJ   Fa0/0: Elect DR 1.1.1.1
*Mar 16 11:48:16.439: OSPF-1 ADJ   Fa0/0: DR: 1.1.1.
R1#1 (Id)   BDR: none 
*Mar 16 11:48:16.439: OSPF-1 LSGEN: Scheduling network LSA on FastEthernet0/0
*Mar 16 11:48:16.443: OSPF-1 ADJ   Fa0/0: Remember old DR 2.2.2.2 (id)
*Mar 16 11:48:16.443: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.443: OSPF-1 ADJ   Fa0/0: 1.1.1.1 address 192.168.12.1 is dead, state DOWN
*Mar 16 11:48:16.447: OSPF-1 ADJ   Fa0/0: Neighbor change event
*Mar 16 11:48:16.447: OSPF-1 ADJ   Fa0/0: DR/BDR election
*Mar 16 11:48:16.447: OSPF-1 ADJ   Fa0/0: Elect BDR 0.0.0.0
*Mar 16 11:48:16.451: OSPF-1 ADJ   Fa0/0: Elect DR 0.0.0.0
*Mar 16 11:48:16.451: OSPF-1 ADJ   Fa0/0: Elect BDR 0.0.0.0
*Mar 16 11:48:16.451: OSPF-1 ADJ   Fa0/0: Elect DR 0.0.0.0
*Mar 16 11:48:16.455: OSPF-1 ADJ   Fa0/0: DR: none    BDR: none 
*Mar 16 11:48:16.455: OSPF-1 ADJ   Fa0/0: Flush network LSA immediately
*Mar 16 11:48:16.459: OSPF-1 ADJ   Fa0/0: Remember old DR 1.1.1.1 (id)
*Mar 16 11:48:16.459: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.459: OS
R1#PF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.467: OSPF-1 ADJ   Lo0: Interface going Up
*Mar 16 11:48:16.467: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 11:48:16.467: OSPF-1 ADJ   Fa0/1: Not sending Hellos until LDP session comes up, FastEthernet0/1 remains down
*Mar 16 11:48:16.471: OSPF-1 ADJ   Fa0/0: Not sending Hellos until LDP session comes up, FastEthernet0/0 remains down
//直到LDP会话建立，否则OSPF不发送Hello消息，F0/0口保持DOWN；
*Mar 16 11:48:16.927: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x80000001
*Mar 16 11:48:16.943: OSPF-1 ADJ   Fa0/0: We are not DR to build Net LSA
*Mar 16 11:48:17.611: OSPF-1 ADJ   Fa0/0: Interface held DOWN waiting for LDP//为了等待LDP会话建立，OSPF保持接口为DOWN；
R1#
*Mar 16 11:48:18.923: OSPF-1 ADJ   Fa0/0: Rcv pkt  src 192.168.12.2 dst 224.0.0.5 id 2.2.2.2 type 5 if_state 0 : ignored due to unknown neighbor
R1#
*Mar 16 11:48:27.611: OSPF-1 ADJ   Fa0/0: Interface held DOWN waiting for LDP
R1#
*Mar 16 11:48:37.611: OSPF-1 ADJ   Fa0/0: Interface held DOWN waiting for LDP
R1#
*Mar 16 11:48:47.611: OSPF-1 ADJ   Fa0/0: Interface held DOWN waiting for LDP//为了等待LDP会话建立，OSPF保持接口为DOWN；
//可以看到LDP-IGP同步机制开始运作了！
//注意：由于LDP会话关系建立在OSPF提供可达性的基础上，而OSPF邻接关系却要等待LDP会话首先建立，
//所以此时OSPF和LDP会互相等待对方完成邻居关系的建立，而导致死循环，最终无论是OSPF还是LDP都无法建立邻居关系；
```

查看OSPF接口F0/0的信息：
```c
R1#show ip ospf interface f0/0
FastEthernet0/0 is up, line protocol is up 
    Internet Address 192.168.12.1/24, Area 0, Attached via Interface Enable
    Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
    Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
    Enabled by interface config, including secondary ip addresses
    Transmit Delay is 1 sec, State DOWN (waiting for LDP), Priority 1
    No designated router on this network
    No backup designated router on this network
    Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
```

配置命令`mpls ldp igp sync holddown msecs`，以告知 IGP 只等待配置命令中所规定的时间，一旦保持计时器超时，不管 LDP 会话是否建立，OSPF 都会建立邻接关系；
```c
R1(config)#mpls ldp igp sync holddown ?
    <1-2147483647>  Hold down time in milliseconds
R1(config)#mpls ldp igp sync holddown 5000
R1(config)#
*Mar 16 12:11:01.743: LDP-SYNC: Fa0/0, OSPF 1: notify status (required, not achieved, delay, holddown 5000) internal status (not achieved, timer not running)
*Mar 16 12:11:01.743: OSPF-1 ADJ   Fa0/0: Rcv int status from LDP: 1 0 1 5000
*Mar 16 12:11:01.747: LDP-SYNC: Fa0/1, OSPF 1: notify status (required, not achieved, delay, holddown 5000) internal status (not achieved, timer not running)
*Mar 16 12:11:01.747: OSPF-1 ADJ   Fa0/1: Rcv int status from LDP: 1 0 1 5000
R1(config)#
*Mar 16 12:11:06.743: OSPF-1 ADJ   Fa0/0: Interface going Up
*Mar 16 12:11:06.743: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 12:11:06.747: OSPF-1 ADJ   Fa0/1: Interface is down
*Mar 16 12:11:06.831: OSPF-1 ADJ   Fa0/0: 2 Way Communication to 2.2.2.2, state 2WAY
*Mar 16 12:11:06.831: OSPF-1 ADJ   Fa0/0: Backup seen event before WAIT timer
*Mar 16 12:11:06.831: OSPF-1 ADJ   Fa0/0: DR/BDR election
*Mar 16 12:11:06.835: OSPF-1 ADJ   Fa0/0: Elect BDR 1.1.1.1
*Mar 16 12:11:06.835: OSPF-1 ADJ   Fa0/0: Elect DR 2.2.2.2
*Mar 16 12:11:06.839: OSPF-1 ADJ   Fa0/0: Elect BDR 1.1.1.1
*Mar 16 12:11:06.839: OSPF-1 ADJ   Fa0/0: Elect DR 2.2.2.2
*Mar 16 12:11:06.839: OSPF-1 ADJ   Fa0/0: DR: 2.2.2.2 (Id)   BDR: 1.1.1.1 (Id)
*Mar 16 12:11:06.843: OSPF-1 ADJ   Fa0/0: Nbr 2.2.2.2: Prepare dbase exchange
*Mar 16 12:11:06.847: OSPF-1 ADJ   Fa0/0: Send DBD to 2.2.2.2 seq 0x2542 opt 0x52 flag 0x7 len 32
*Mar 16 12:11:06.847: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 12:11:06
R1(config)#.891: OSPF-1 ADJ   Fa0/0: Rcv DBD from 2.2.2.2 seq 0x1A5A opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Mar 16 12:11:06.891: OSPF-1 ADJ   Fa0/0: NBR Negotiation Done. We are the SLAVE
*Mar 16 12:11:06.895: OSPF-1 ADJ   Fa0/0: Nbr 2.2.2.2: Summary list built, size 1
*Mar 16 12:11:06.895: OSPF-1 ADJ   Fa0/0: Send DBD to 2.2.2.2 seq 0x1A5A opt 0x52 flag 0x2 len 52
*Mar 16 12:11:06.963: OSPF-1 ADJ   Fa0/0: Rcv DBD from 2.2.2.2 seq 0x1A5B opt 0x52 flag 0x1 len 52  mtu 1500 state EXCHANGE
*Mar 16 12:11:06.967: OSPF-1 ADJ   Fa0/0: Exchange Done with 2.2.2.2
*Mar 16 12:11:06.967: OSPF-1 ADJ   Fa0/0: Send LS REQ to 2.2.2.2 length 12 LSA count 1
*Mar 16 12:11:06.967: OSPF-1 ADJ   Fa0/0: Send DBD to 2.2.2.2 seq 0x1A5B opt 0x52 flag 0x0 len 32
*Mar 16 12:11:07.039: OSPF-1 ADJ   Fa0/0: Rcv LS UPD from 2.2.2.2 length 76 LSA count 1
*Mar 16 12:11:07.039: OSPF-1 ADJ   Fa0/0: Synchronized with 2.2.2.2, state FULL
*Mar 16 12:11:07.043: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEtherne
R1(config)#t0/0 from LOADING to FULL, Loading Done
*Mar 16 12:11:07.043: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 12:11:07.047: OSPF-1 ADJ   Fa0/0: Rcv LS REQ from 2.2.2.2 length 36 LSA count 1
*Mar 16 12:11:07.243: OSPF-1 LSGEN: Build router LSA for area 0, router ID 1.1.1.1, seq 0x80000002 //注意此LSA的序列号；
R1(config)#
*Mar 16 12:11:09.979: OSPF-1 ADJ   Fa0/0: Neighbor change event
*Mar 16 12:11:09.983: OSPF-1 ADJ   Fa0/0: DR/BDR election
*Mar 16 12:11:09.983: OSPF-1 ADJ   Fa0/0: Elect BDR 1.1.1.1
*Mar 16 12:11:09.987: OSPF-1 ADJ   Fa0/0: Elect DR 2.2.2.2
*Mar 16 12:11:09.987: OSPF-1 ADJ   Fa0/0: DR: 2.2.2.2 (Id)   BDR: 1.1.1.1 (Id)
*Mar 16 12:11:09.991: OSPF-1 LSGEN: Scheduling rtr LSA for area 0
*Mar 16 12:11:10.491: OSPF-1 LSGEN: Rate limit LSA generation for 1.1.1.1 1.1.1.1 1
R1(config)#
*Mar 16 12:11:12.243: OSPF-1 LSGEN: No change in router LSA, area 0 
//当holdtime超时后，虽然LDP会话还没有建立，OSPF却主动建立了邻接关系；
```

查看R1的OSPF Database中1类LSA；
```c
R1#show ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

    LS age: 304
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 1.1.1.1
    Advertising Router: 1.1.1.1
    LS Seq Number: 80000002
    Checksum: 0x30F7
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 1.1.1.1
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.1
        Number of MTID metrics: 0
        TOS 0 Metrics: 65535 //接口为UP，OSPF正在发送最大度量值；

    LS age: 304
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 2.2.2.2
    Advertising Router: 2.2.2.2
    LS Seq Number: 80000059
    Checksum: 0x8F33
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 2.2.2.2
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.2
        Number of MTID metrics: 0
```

查看R1上F0/0的LDP和OSPF同步信息；
```c
//备注：由于holdtime为5秒（5000毫秒），所以输入下一条命令，必须非常快！
R1(config)#do sh ip ospf mpls ldp interface
Loopback0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Not required
    Holddown timer is disabled
    Interface is up 
FastEthernet0/1
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs
    Holddown timer is not running
    Interface is down 
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs //LDP-IGP同步保持计时器被配置为5000毫秒；
    Holddown timer is running and is expiring in 1848 msecs//LDP-IGP保持计时器正在运行，并且将在1848毫秒后到期超时；
    Interface is down and pending LDP //接口为Down，等待LDP；
```

查看R1上F0/0的LDP和OSPF同步信息；
```c
R1(config)#do sh ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs
    Holddown timer is not running//LDP-IGP保持计时器未在运行，说明保持计时器已经超时；
    Interface is up and sending maximum metric//接口为UP，OSPF正在发送最大度量值；
```
> 可以看到虽然OSPF邻接关系已经建立，但是由于LDP会话还没有建立，所以OSPF向邻居通告的度量值为最大值65535，在具有多条链路同时能够达到目的地的情况下，这一OSPF的这一动作确保此链路不会被用来转发数据；从而防止了到达该此链路的数据包不会因不能被打上标签，而导致数据包被丢弃；
         

在R2上，将F0/0口的mpls开启：
```c
R2(config-if)#int f0/0 
R2(config-if)#mpls ip

*Mar 16 12:29:53.575: %LDP-5-NBRCHG: LDP Neighbor 1.1.1.1:0 (1) is UP
//可以看到LDP邻居马上建立；
```

查看R1的OSPF Database中1类LSA：
```c
R1#sh ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

    LS age: 122
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 1.1.1.1
    Advertising Router: 1.1.1.1
    LS Seq Number: 80000003 //R1的OSPF进程更新了之前的Seq：80000002的LSA；
    Checksum: 0x4CD9
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 1.1.1.1
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.1
        Number of MTID metrics: 0
        TOS 0 Metrics: 1 //在更新后的Seq：80000003 LSA中，度量值被更改回正常的1；

    LS age: 1250
    Options: (No TOS-capability, DC)
    LS Type: Router Links
    Link State ID: 2.2.2.2
    Advertising Router: 2.2.2.2
    LS Seq Number: 80000059
    Checksum: 0x8F33
    Length: 48
    Number of Links: 2

    Link connected to: a Stub Network
        (Link ID) Network/subnet number: 2.2.2.2
        (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
        TOS 0 Metrics: 1

    Link connected to: a Transit Network
        (Link ID) Designated Router address: 192.168.12.2
        (Link Data) Router Interface address: 192.168.12.2
        Number of MTID metrics: 0
        TOS 0 Metrics: 1
```

查看R1上F0/0的LDP和OSPF同步信息；
```c
R1#sh ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs
    Holddown timer is not running
    Interface is up
//可以看到接口已经起来了，说明LDP和OSPF已经同步；
```


## 实验2：R1和R2之间仅有两条或多条链路时的LDP-IGP同步

将R1和R2之间的由F0/1相连的链路启用，让R1和R2之间形成双链路；
```c
R1(config)#int f0/1
R1(config-if)#no shutdown

R2(config)#int f0/1
R2(config-if)#no shutdown
```

查看R1的LDP邻居表；
```c
R1#show mpls ldp neighbor
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 2.2.2.2.52561 - 1.1.1.1.646
        State: Oper; Msgs sent/rcvd: 15/15; Downstream
        Up time: 00:06:36
        LDP discovery sources:
            FastEthernet0/0, Src IP addr: 192.168.12.2
            FastEthernet0/1, Src IP addr: 192.168.21.2
        Addresses bound to peer LDP Ident:
            2.2.2.2         192.168.12.2    192.168.21.2 
//可以看到，R1在F0/0和F0/1发现了LDP邻居；
```

禁用R2上F0/0的MPLS，让R1和R2的LDP邻居关系断开，确保LDP会话晚于OSPF邻接关系建立；
```c
R2(config-if)#int f0/0  
R2(config-if)#no mpls ip
```

此时，在经过将R2的F0/0禁用MPLS之后，R1和R2的LDP邻居已经关系。接下来，重置OSPF进程：
```c
R1#clear mpls ldp neighbor *
R1# 
*Mar 16 13:53:14.798: %LDP-5-CLEAR_NBRS: Clear LDP neighbors (*) by console
*Mar 16 13:53:14.810: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (2) is DOWN (User cleared session manually)
*Mar 16 13:53:19.914: %LDP-5-NBRCHG: LDP Neighbor 2.2.2.2:0 (1) is UP
R1#clear ip ospf process
Reset ALL OSPF processes? [no]: yes
*Mar 16 13:53:27.098: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
*Mar 16 13:53:27.102: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Mar 16 13:53:27.442: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/1 from LOADING to FULL, Loading Done
//备注：快速输入命令查看R1接口F0/0的LDP与OSPF的同步信息；
R1#sh ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs//LDP-IGP同步保持计时器被配置为5000毫秒；
    Holddown timer is running and is expiring in 952 msecs//LDP-IGP保持计时器正在运行，并且将在952毫秒后到期超时；
    Interface is down and pending LDP
//备注：快速输入命令查看R1接口F0/0的LDP与OSPF的同步信息；
R1#sh ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs
    Holddown timer is not running//LDP-IGP保持计时器未在运行，说明保持计时器已经超时；
    Interface is up and sending maximum metric//OSPF已经让接口UP，但却在发送最大度量值；
//可以看到虽然OSPF邻接关系已经建立，但是由于LDP会话还没有建立，
//所以OSPF向邻居通告的度量值为最大值65535，在具有多条链路同时能够达到目的地的情况下，
//这一OSPF的这一动作确保此链路不会被用来转发数据；
//从而防止了到达该此链路的数据包不会因不能被打上标签，而导致数据包被丢弃；
```

查看R1的OSPF邻居表：
```c
R1#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:31    192.168.21.2    FastEthernet0/1
2.2.2.2           1   FULL/DR         00:00:32    192.168.12.2    FastEthernet0/0
//可以看到R1仍然从两个接口发现了OSPF邻居R2；
```

查看R1的MPLS LDP邻居表；
```c
R1#show mpls ldp neighbor
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 2.2.2.2.64074 - 1.1.1.1.646
        State: Oper; Msgs sent/rcvd: 19/19; Downstream
        Up time: 00:11:03
        LDP discovery sources:
            FastEthernet0/1, Src IP addr: 192.168.21.2
        Addresses bound to peer LDP Ident:
            2.2.2.2         192.168.12.2    192.168.21.2 
//可以看到R1仅仅从F0/1发现了LDP邻居R2；
```

查看R1的LFIB表；
```c
R1#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  2.2.2.2/32       0             Fa0/1      192.168.21.2
//可以看到仅有从F0/1学到标签；
```

查看R1的IP路由表；
```c
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
                                                            
Gateway of last resort is not set
                                                            
        1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
        2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/2] via 192.168.21.2, 00:24:13, FastEthernet0/1//虽然从两条链路学到路由，但是OSPF与LDP保持了一致，只使用其中走F0/1的路径；
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.1/32 is directly connected, FastEthernet0/0
        192.168.21.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.21.0/24 is directly connected, FastEthernet0/1
L        192.168.21.1/32 is directly connected, FastEthernet0/1
//注意：本来我以为虽然LFIB中只有走F0/1的一条出站路径，而路由表中会有两条OSPF路由，分别是从F0/0和F0/1学到的，但事实并非如此；
//正是因为开启了LDP-IGP同步，所以造成了这样的结果；
```

启用R2 f0/0的MPLS；
```c
R2(config)#int f0/0
R2(config-if)#mpls ip
```

查看R1上接口F0/0的LDP-OSPF同步信息；
```c
R1(config-router)#do sh ip ospf mpls ldp interface f0/0
FastEthernet0/0
    Process ID 1, Area 0
    LDP is not configured through LDP autoconfig
    LDP-IGP Synchronization : Required
    Holddown timer is configured : 5000 msecs
    Holddown timer is not running
    Interface is up
//注意：当R2的F0/0启用了MPLS后，R1的OSPF将接口F0/0度量值还原为了正常值；
```

查看R1的LFIB表；
```c
R1(config-router)#do sh mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  2.2.2.2/32       0             Fa0/0      192.168.12.2
            Pop Label  2.2.2.2/32       0             Fa0/1      192.168.21.2
//当R2的F0/0启用了MPLS后，R1具有了两条等价的去往2.2.2.2的路径；
```

查看R1的IP路由表；
```c
R1(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override

Gateway of last resort is not set

        1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
        2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/2] via 192.168.21.2, 00:47:46, FastEthernet0/1//当LDP从F0/0和F0/1收到两条标签绑定信息后，路由表中同样有了两条路径；
                        [110/2] via 192.168.12.2, 00:20:20, FastEthernet0/0
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.1/32 is directly connected, FastEthernet0/0
        192.168.21.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.21.0/24 is directly connected, FastEthernet0/1
L        192.168.21.1/32 is directly connected, FastEthernet0/1
```  
            
验证关闭LDP-IGP同步后LFIB和IP路由表情况；
```c
//关闭LDP-IGP同步；
R1(config)#router os 1
R1(config-router)#no mpls ldp sync

R2(config)#router os 1
R2(config-router)#no mpls ldp sync

//禁用R2上F0/0的MPLS；
R2(config-router)#int f0/0
R2(config-if)#no mpls ip
```

查看R1的LIB表：
```c
R1#show mpls ip binding
    1.1.1.1/32 
        in label:     imp-null  
        out label:    16        lsr: 2.2.2.2:0       
    2.2.2.2/32 
        in label:     16        
        out label:    imp-null  lsr: 2.2.2.2:0        inuse
    192.168.12.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 2.2.2.2:0       
    192.168.21.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 2.2.2.2:0
```
                
查看R1的LFIB表：
```c
R1#sh mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  2.2.2.2/32       0             Fa0/1      192.168.21.2
                No Label   2.2.2.2/32       0             Fa0/0      192.168.12.2
```

查看R1的IP路由表；
```c
R1#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override

Gateway of last resort is not set

        1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
        2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/2] via 192.168.21.2, 00:54:25, FastEthernet0/1
                        [110/2] via 192.168.12.2, 00:26:59, FastEthernet0/0
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.1/32 is directly connected, FastEthernet0/0
        192.168.21.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.21.0/24 is directly connected, FastEthernet0/1
L        192.168.21.1/32 is directly connected, FastEthernet0/1
//注意：关闭LDP-IGP同步后，即使没有从F0/0口收到R2通告的标签，R1的路由表中还是出现两条去往2.2.2.2/32的等价路径；
//如果这种情况出现是在MPLS VPN，AToM 或 VPLS环境中P路由器上，可以能会出现丢包现象；
```