---
title: MPLS 实验4：MPLS vs BGP同步
date: 2014-03-14 22:46:41
updated: 2014-03-14 22:46:41
tags: [MPLS, Network]
categories: Network
---


# 实验环境：
- 模拟器：GNS3 0.8.6
- Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image

GNS3实验拓扑文件：
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
mpls label protocol ldp
!
interface Loopback0
    ip address 1.1.1.1 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.12.1 255.255.255.0
    ip ospf 1 area 0
    no shut
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.15.1 255.255.255.0
    no shut
!
router ospf 1
    router-id 1.1.1.1
!
router bgp 1
    bgp router-id 1.1.1.1
    bgp log-neighbor-changes
    network 1.1.1.1 mask 255.255.255.255
    neighbor 4.4.4.4 remote-as 1
    neighbor 4.4.4.4 update-source Loopback0
    neighbor 4.4.4.4 next-hop-self
    neighbor 192.168.15.5 remote-as 2
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
    ip address 192.168.23.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
router ospf 1
    router-id 2.2.2.2
!
mpls ldp router-id Loopback0 force
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R3:
```
hostname R3
!
ip cef
!
mpls label protocol ldp
!
interface Loopback0
    ip address 3.3.3.3 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.34.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.23.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
router ospf 1
    router-id 3.3.3.3
!
mpls ldp router-id Loopback0 force
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R4:
```
hostname R4
!
ip cef
!
mpls label protocol ldp
!
interface Loopback0
    ip address 4.4.4.4 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.34.4 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.47.4 255.255.255.0
    no shutdown
!
router ospf 1
    router-id 4.4.4.4
!
router bgp 1
    bgp router-id 4.4.4.4
    bgp log-neighbor-changes
    network 4.4.4.4 mask 255.255.255.255
    neighbor 1.1.1.1 remote-as 1
    neighbor 1.1.1.1 update-source Loopback0
    neighbor 1.1.1.1 next-hop-self
    neighbor 192.168.47.7 remote-as 3
!
mpls ldp router-id Loopback0 force
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

### R5:
```
hostname R5
!
ip cef
!
interface Loopback0
    ip address 5.5.5.5 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.56.5 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.15.5 255.255.255.0
    no shutdown
!
router ospf 1
    router-id 5.5.5.5
!
router bgp 2
    bgp router-id 5.5.5.5
    bgp log-neighbor-changes
    network 5.5.5.5 mask 255.255.255.255
    neighbor 192.168.15.1 remote-as 1
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R6:
```
hostname R6
!
ip cef
!
interface Loopback0
    ip address 6.6.6.6 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.56.6 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
router ospf 1
    router-id 6.6.6.6
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R7:
```
hostname R7
!
ip cef
!
interface Loopback0
    ip address 7.7.7.7 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.78.7 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.47.7 255.255.255.0
    no shutdown
!
router ospf 1
    router-id 7.7.7.7
!
router bgp 3
    bgp router-id 7.7.7.7
    bgp log-neighbor-changes
    network 7.7.7.7 mask 255.255.255.255
    neighbor 192.168.47.4 remote-as 1
!
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

## R8:
```
hostname R8
!
ip cef
!
interface Loopback0
    ip address 8.8.8.8 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/0
    ip address 192.168.78.8 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
router ospf 1
    router-id 8.8.8.8
!         
line con 0
    exec-timeout 0 0
    logging synchronous
!
end
```

# 实验目的：
1. 使用MPLS解决BGP关闭同步后IBGP非全互联造成的路由黑洞问题；
2. 研究以递归方式为前缀分配标签是如何运作的；

# 实验与调试：
## 实验1：
验证：从R5上，以R5的Loopback做源ping R7的Loopback0；
```c
R5#ping 7.7.7.7 source l0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 7.7.7.7, timeout is 2 seconds:
Packet sent with a source address of 5.5.5.5 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 112/124/140 ms
//成功了！
```

**问题：**
- 在该实验中，R2和R3作为P路由器，并没有运行BGP，为什么以R5的Loopback0做源可以ping通R7的Loopback0？
- 为什么数据包R2和R3在没有5.5.5.5/32和7.7.7.7/32的路由的情况下没有丢弃包？

模拟ping包的交换过程，假设此时R1收到了来自R5的ping包：
> 注意：当数据包进入R1时，数据包并未携带标签，此时R1作为LSP中的第一台LSR，也是入站LSR，R1需要查找CEF来为这个标准的IP数据包打上标签；

查看R1的路由表：
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
O        2.2.2.2 [110/2] via 192.168.12.2, 01:20:09, FastEthernet0/0
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/3] via 192.168.12.2, 01:20:09, FastEthernet0/0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/4] via 192.168.12.2, 01:20:09, FastEthernet0/0
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.15.5, 00:17:45
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [200/0] via 4.4.4.4, 00:16:46
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.1/32 is directly connected, FastEthernet0/0
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.1/32 is directly connected, FastEthernet0/1
O     192.168.23.0/24 [110/2] via 192.168.12.2, 01:20:09, FastEthernet0/0
O     192.168.34.0/24 [110/3] via 192.168.12.2, 01:20:09, FastEthernet0/0
//在R1上，路由表递归查找下一跳可以发现：要去往7.7.7.7/32必须先到达4.4.4.4；
//要到达4.4.4.4/32必须先到达192.168.12.2；
```

查看R1的LFIB表：
```c
R1#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         16         4.4.4.4/32       0             Fa0/0      192.168.12.2
17         17         3.3.3.3/32       0             Fa0/0      192.168.12.2
18         Pop Label  2.2.2.2/32       0             Fa0/0      192.168.12.2
19         18         192.168.34.0/24  0             Fa0/0      192.168.12.2
20         Pop Label  192.168.23.0/24  0             Fa0/0      192.168.12.2
//由LFIB表可以得出，去往前缀4.4.4.4/32的出标签为16；
```

沿路由递归查看CEF表：
```c
R1#sh ip cef 7.7.7.7 detail
7.7.7.7/32, epoch 0, flags rib only nolabel, rib defined all labels
    recursive via 4.4.4.4
    nexthop 192.168.12.2 FastEthernet0/0 label 16

R1#sh ip cef 4.4.4.4 detail
4.4.4.4/32, epoch 0
    local label info: global/16
    1 RR source [no flags]
    nexthop 192.168.12.2 FastEthernet0/0 label 16
//由于路由递归查找的关系，去往前缀7.7.7.7/32的数据包在出站时被打上了和去往前缀4.4.4.4/32的数据包相同的出标签16（R1的出标签，R2的入标签）；
```

```
R1#sh ip cef 192.168.12.2 detail
192.168.12.2/32, epoch 0, flags attached
    Adj source: IP adj out of FastEthernet0/0, addr 192.168.12.2 6819DAE0
    Dependent covered prefix type adjfib cover 192.168.12.0/24
    attached to FastEthernet0/0
//由于R1与192.168.12.0/24直连，所以不再需要在进行标签转发，所以R1没有关于192.168.12.0/24的出标签；
```

**此时数据达到了R2：**

查看R2的IP路由表：
```c
R2#show ip route
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
O        1.1.1.1 [110/2] via 192.168.12.1, 03:42:35, FastEthernet0/0
        2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback0
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/2] via 192.168.23.3, 03:42:45, FastEthernet0/1
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/3] via 192.168.23.3, 03:42:45, FastEthernet0/1
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.2/32 is directly connected, FastEthernet0/0
        192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.23.0/24 is directly connected, FastEthernet0/1
L        192.168.23.2/32 is directly connected, FastEthernet0/1
O     192.168.34.0/24 [110/2] via 192.168.23.3, 03:42:45, FastEthernet0/1
//可以看到，由于R2没有运行BGP，所以R2的路由表中没有关于前缀7.7.7.7/32的路由条目；
//注意：递归查找路由表只会发生在运行BGP并与其他PE路由器建立IBGP关系的PE路由器，
//而在P路由器上，由于没有运行BGP，不可能有相关路由信息，所以只按照标签转发。
```

查看R2的LFIB：
```c
R2#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         16         4.4.4.4/32       38689         Fa0/1      192.168.23.3
17         Pop Label  3.3.3.3/32       0             Fa0/1      192.168.23.3
18         Pop Label  192.168.34.0/24  686           Fa0/1      192.168.23.3
19         Pop Label  1.1.1.1/32       41711         Fa0/0      192.168.12.1
//尽管R2没有关于前缀7.7.7.7/32的路由条目，但由于AS1是MPLS域，路由器都使用标签转发数据，
//加之因为路由的递归查找，使得前缀7.7.7.7/32“继承了”前缀4.4.4.4/32的标签；
//所以，当R2收到去往7.7.7.7/32的数据包时，此数据包携带了前缀4.4.4.4/32的入标签16（R2的入标签，R1的出标签）；
//于是，R2作出转发决定，并给数据包打上出标签16（R2的出标签，R3的入标签），然后转发给R3；
```

**此时数据到达了R3：**

查看R3的IP路由表：
```c
R3#show ip route
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
O        1.1.1.1 [110/3] via 192.168.23.2, 03:55:45, FastEthernet0/1
        2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/2] via 192.168.23.2, 03:55:55, FastEthernet0/1
        3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/2] via 192.168.34.4, 03:56:35, FastEthernet0/0
O     192.168.12.0/24 [110/2] via 192.168.23.2, 03:55:55, FastEthernet0/1
        192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.23.0/24 is directly connected, FastEthernet0/1
L        192.168.23.3/32 is directly connected, FastEthernet0/1
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, FastEthernet0/0
L        192.168.34.3/32 is directly connected, FastEthernet0/0
//同样的，由于R2没有运行BGP，所以R3的路由表中没有关于前缀7.7.7.7/32的路由条目；
```

查看R3的LFIB表：
```c
R3#sh mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  4.4.4.4/32       35742         Fa0/0      192.168.34.4
17         Pop Label  2.2.2.2/32       0             Fa0/1      192.168.23.2
18         Pop Label  192.168.12.0/24  1064          Fa0/1      192.168.23.2
19         19         1.1.1.1/32       43006         Fa0/1      192.168.23.2
//尽管R2没有关于前缀7.7.7.7/32的路由条目，但由于AS1是MPLS域，路由器都使用标签转发数据，
//加之因为路由的递归查找，使得前缀7.7.7.7/32“继承了”前缀4.4.4.4/32的标签；
//所以，当R3收到去往7.7.7.7/32的数据包时，此数据包携带了前缀4.4.4.4/32的入标签16（R3的入标签，R2的出标签）；
//于是，R3作出转发决定，并将数据包的标签弹出（R3的出标签为隐式空标签，R4的通告给R3的入标签为隐式空标签），然后转发给R4；
```

**此时数据到达了R4:**

由于R4是LSP中的最后一台LSR，即出站LSR，所以R4负责移除标签，然后查找CEF，按照IP数据包的标准流程进行处理；

查看R4的LFIB表：
```c
R4#sh mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  3.3.3.3/32       0             Fa0/0      192.168.34.3
17         Pop Label  192.168.23.0/24  0             Fa0/0      192.168.34.3
18         17         2.2.2.2/32       0             Fa0/0      192.168.34.3
19         19         1.1.1.1/32       0             Fa0/0      192.168.34.3
20         18         192.168.12.0/24  0             Fa0/0      192.168.34.3
```           
                
查看R4的IP路由表：
```c
R4#show ip route
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
O        1.1.1.1 [110/4] via 192.168.34.3, 04:16:48, FastEthernet0/0
        2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/3] via 192.168.34.3, 04:16:48, FastEthernet0/0
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/2] via 192.168.34.3, 04:17:36, FastEthernet0/0
        4.0.0.0/32 is subnetted, 1 subnets
C        4.4.4.4 is directly connected, Loopback0
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [200/0] via 1.1.1.1, 03:14:32
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [20/0] via 192.168.47.7, 03:13:34
O     192.168.12.0/24 [110/3] via 192.168.34.3, 04:16:48, FastEthernet0/0
O     192.168.23.0/24 [110/2] via 192.168.34.3, 04:17:36, FastEthernet0/0
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, FastEthernet0/0
L        192.168.34.4/32 is directly connected, FastEthernet0/0
        192.168.47.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.47.0/24 is directly connected, FastEthernet0/1
L        192.168.47.4/32 is directly connected, FastEthernet0/1
```

查看R4的CEF表：
```c
R4#show ip cef 7.7.7.7 detail
7.7.7.7/32, epoch 0, flags rib only nolabel, rib defined all labels
    recursive via 192.168.47.7
    attached to FastEthernet0/1

//至此，ping包被按照普通IP数据包转发给了R7，echo reply包的转发交换方向相反，但过程相同；
```


# 实验结论：
1. 在分配标签时，标签可以以路由递归的方式被继承；
2. MPLS 不但可以解决BGP关闭同步后需要IBGP全互联的问题，更高效的是，MPLS可以让MPLS域内的P路由器不必运行BGP；
