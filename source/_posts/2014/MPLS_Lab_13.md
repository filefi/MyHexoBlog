---
title: MPLS 实验13：Hub-and-Spoke
date: 2014-03-19 23:59:51
updated: 2014-03-19 23:59:51
tags: [MPLS, Network]
categories: Network
---


<!-- more -->


# 实验环境：
- 模拟器：GNS3 0.8.6
- Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image

# GNS3实验拓扑文件：
[拓扑文件](topology.net)

# 实验拓扑：
![](topo.png)


# 实验场景：
1. Spoke站点只能与Hub站点进行通信；
2. Spoke站点到Spoke站点的流量必须首先发送到Hub站点；
    
要实现以上操作，就必须要使用两个不同的RT，分别负责导入和导出；还需要为不同站点分配不同的RD；

1. 使用RT控制VPNv4路由的导入导出；
2. 使用在PE和CE使用BGP Allowas-in解决eBGP防环机制导致
3. 使用SOO解决PE-CE路由协议eBGP的路由环路；


# 基本预配置：
## R1：
```c
hostname R1
!
ip cef
!
ip vrf A-Site1
        rd 1:1
        route-target export 1:1
        route-target import 1:2
!
mpls label protocol ldp
!
interface Loopback0
        ip address 1.1.1.1 255.255.255.255
        ip ospf 1 area 0
!
interface FastEthernet0/0
        ip address 192.168.13.1 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet0/1
        ip vrf forwarding A-Site1
        ip address 192.168.15.1 255.255.255.0
        no shutdown
!
router ospf 1
        router-id 1.1.1.1
!
router bgp 1
        bgp router-id 1.1.1.1
        bgp log-neighbor-changes
        neighbor ibgp peer-group
        neighbor ibgp remote-as 1
        neighbor ibgp update-source Loopback0
        neighbor ibgp next-hop-self
        neighbor 2.2.2.2 peer-group ibgp
        neighbor 4.4.4.4 peer-group ibgp
        !
        address-family vpnv4
        neighbor ibgp send-community both
        neighbor 2.2.2.2 activate
        neighbor 4.4.4.4 activate
        exit-address-family
        !
        address-family ipv4 vrf A-Site1
        neighbor 192.168.15.5 remote-as 65001
        neighbor 192.168.15.5 activate
        redistribute connected
        exit-address-family
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
```c
hostname R2
!
ip cef
!
ip vrf A-Site1
        rd 1:1
        route-target export 1:1
        route-target import 1:2
!
ip vrf A-Site2
        rd 1:2
        route-target export 1:1
        route-target import 1:2
!
mpls label protocol ldp
!
interface Loopback0
        ip address 2.2.2.2 255.255.255.255
        ip ospf 1 area 0
!
interface FastEthernet0/0
        ip vrf forwarding A-Site1
        ip address 192.168.25.2 255.255.255.0
        no shutdown
!
interface FastEthernet0/1
        ip address 192.168.23.2 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet1/0
        ip vrf forwarding A-Site2
        ip address 192.168.26.2 255.255.255.0
        no shutdown
!
router ospf 1
        router-id 2.2.2.2
!
router bgp 1
        bgp log-neighbor-changes
        neighbor ibgp peer-group
        neighbor ibgp remote-as 1
        neighbor ibgp update-source Loopback0
        neighbor ibgp next-hop-self
        neighbor 1.1.1.1 peer-group ibgp
        neighbor 4.4.4.4 peer-group ibgp
        !
        address-family vpnv4
        neighbor ibgp send-community both
        neighbor 1.1.1.1 activate
        neighbor 4.4.4.4 activate
        exit-address-family
        !
        address-family ipv4 vrf A-Site1
        redistribute connected
        neighbor 192.168.25.5 remote-as 65001
        neighbor 192.168.25.5 activate
        exit-address-family
        !
        address-family ipv4 vrf A-Site2
        redistribute connected
        neighbor 192.168.26.6 remote-as 65002
        neighbor 192.168.26.6 activate
        exit-address-family
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
```c
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
        ip address 192.168.13.3 255.255.255.0
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
interface FastEthernet1/0
        ip address 192.168.34.3 255.255.255.0
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
```c
hostname R4
!
ip cef
!
ip vrf A-Site3
        rd 1:3
        route-target export 1:2
        route-target import 1:1
!
mpls label protocol ldp
!
interface Loopback0
        ip address 4.4.4.4 255.255.255.255
        ip ospf 1 area 0
!
interface FastEthernet0/0
        ip vrf forwarding A-Site3
        ip address 192.168.47.4 255.255.255.0
        no shutdown
!
interface FastEthernet0/1
        ip vrf forwarding A-Site3
        ip address 192.168.48.4 255.255.255.0
        no shutdown
!
interface FastEthernet1/0
        ip address 192.168.34.4 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
router ospf 1
        router-id 4.4.4.4
!
router bgp 1
        bgp log-neighbor-changes
        neighbor ibgp peer-group
        neighbor ibgp remote-as 1
        neighbor ibgp update-source Loopback0
        neighbor ibgp next-hop-self
        neighbor 1.1.1.1 peer-group ibgp
        neighbor 2.2.2.2 peer-group ibgp
        !
        address-family vpnv4
        neighbor ibgp send-community both
        neighbor 1.1.1.1 activate
        neighbor 2.2.2.2 activate
        exit-address-family
        !
        address-family ipv4 vrf A-Site3
        neighbor 192.168.47.7 remote-as 65003
        neighbor 192.168.47.7 activate
        neighbor 192.168.48.8 remote-as 65003
        neighbor 192.168.48.8 activate
        redistribute connected
        exit-address-family
!
mpls ldp router-id Loopback0 force
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
        
## R5:
```c
hostname R5
!
ip cef
!
interface Loopback0
        ip address 5.5.5.5 255.255.255.255
!
interface FastEthernet0/0
        ip address 192.168.25.5 255.255.255.0
        no shutdown
!
interface FastEthernet0/1
        ip address 192.168.15.5 255.255.255.0
        no shutdown
!
router bgp 65001
        bgp log-neighbor-changes
        network 5.5.5.5 mask 255.255.255.255
        neighbor 192.168.15.1 remote-as 1
        neighbor 192.168.25.2 remote-as 1
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
        
## R6:
```c
hostname R6
!
ip cef
!
interface Loopback0
        ip address 6.6.6.6 255.255.255.255
!
interface FastEthernet0/0
        ip address 192.168.26.6 255.255.255.0
        no shutdown
!
router bgp 65002
        bgp log-neighbor-changes
        network 6.6.6.6 mask 255.255.255.255
        neighbor 192.168.26.2 remote-as 1
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
        
## R7:
```c
hostname R7
!
ip cef
!
interface Loopback0
        ip address 7.7.7.7 255.255.255.255
!
interface FastEthernet0/0
        ip address 192.168.47.7 255.255.255.0
        no shutdown
!
router bgp 65003
        bgp log-neighbor-changes
        network 7.7.7.7 mask 255.255.255.255
        neighbor 8.8.8.8 remote-as 65003
        neighbor 8.8.8.8 update-source Loopback0
        neighbor 8.8.8.8 next-hop-self
        neighbor 192.168.47.4 remote-as 1
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
        
## R8:
```c
hostname R8
!
ip cef
!
interface Loopback0
        ip address 8.8.8.8 255.255.255.255
!
interface FastEthernet0/1
        ip address 192.168.48.8 255.255.255.0
        no shutdown
!
router bgp 65003
        bgp log-neighbor-changes
        network 8.8.8.8 mask 255.255.255.255
        neighbor 7.7.7.7 remote-as 65003
        neighbor 7.7.7.7 update-source Loopback0
        neighbor 7.7.7.7 next-hop-self
        neighbor 192.168.48.4 remote-as 1
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```


# 实验与调试：
## 实验1：同一VPN的不同站点使用不同的AS号；
查看R7的BGP汇总表；
```c
R7(config-router)#do sh ip bgp summary
BGP router identifier 7.7.7.7, local AS number 65003
BGP table version is 12, main routing table version 12
5 network entries using 680 bytes of memory
5 path entries using 280 bytes of memory
4/4 BGP path/bestpath attribute entries using 512 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1544 total bytes of memory
BGP activity 6/1 prefixes, 16/11 paths, scan interval 60 secs
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
8.8.8.8         4        65003       0       0        1    0    0 00:04:50 Idle
192.168.47.4    4            1     107      87       12    0    0 01:11:52        4
//注意到邻居R8处于Idle状态；
```

查看R8的BGP汇总表；
```c
R8(config-router)#do sh ip bgp summary
BGP router identifier 8.8.8.8, local AS number 65003
BGP table version is 12, main routing table version 12
5 network entries using 680 bytes of memory
5 path entries using 280 bytes of memory
4/4 BGP path/bestpath attribute entries using 512 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1544 total bytes of memory
BGP activity 6/1 prefixes, 14/9 paths, scan interval 60 secs
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
7.7.7.7         4        65003       0       0        1    0    0 00:07:55 Idle
192.168.48.4    4            1     101      92       12    0    0 01:14:12        4
//注意到邻居R7处于Idle状态；
```

查看R7的路由表；
```c
R7(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.47.4, 00:54:25
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.47.4, 00:40:41
        7.0.0.0/32 is subnetted, 1 subnets
C        7.7.7.7 is directly connected, Loopback0
B     192.168.25.0/24 [20/0] via 192.168.47.4, 00:41:31
B     192.168.26.0/24 [20/0] via 192.168.47.4, 00:41:06
        192.168.47.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.47.0/24 is directly connected, FastEthernet0/0
L        192.168.47.7/32 is directly connected, FastEthernet0/0
//注意到R7的没有去往BGP邻居R8环回口的路由，所以它们无法建立邻居；
```

查看R8的路由表；
```c
R8(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.48.4, 01:06:21
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.48.4, 00:53:21
        8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback0
B     192.168.25.0/24 [20/0] via 192.168.48.4, 00:54:12
B     192.168.26.0/24 [20/0] via 192.168.48.4, 00:53:47
        192.168.48.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.48.0/24 is directly connected, FastEthernet0/1
L        192.168.48.8/32 is directly connected, FastEthernet0/1
//注意到R8的没有去往BGP邻居R7环回口的路由，所以它们无法建立邻居；
```

由于R7和R8同属于一个AS，但它们被PE3隔离开了，并且它们之间再没有其他链路可以到达对方，因此，PE3为R7和R8提供到达对方的连通性，但由于EBGP的AS_PATH环路防护机制，
导致R7和R8从PE3收到包含自己所在AS的EBGP路由之后，将直接丢弃这些路由，所以R7和R8不会有对方的路由；
 
配置 `allowas-in` ，使R7和R8放松对AS_PATH的环路防护检测；
```c
R7(config-router)#nei 192.168.47.4 allowas-in 2
```
```c
R8(config-router)#nei 192.168.48.4 allowas-in 2
```

查看R7的BGP汇总表；
```c
R7(config-router)#do sh ip bgp summary
BGP router identifier 7.7.7.7, local AS number 65003
BGP table version is 22, main routing table version 22
6 network entries using 816 bytes of memory
13 path entries using 728 bytes of memory
10/5 BGP path/bestpath attribute entries using 1280 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2920 total bytes of memory
BGP activity 7/1 prefixes, 31/18 paths, scan interval 60 secs
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
8.8.8.8         4        65003       9      13       22    0    0 00:01:54        6
192.168.47.4    4            1     148     125       22    0    0 01:44:39        6
```

查看R8的BGP汇总表；
```c
R8(config-router)#do sh ip bgp summary
BGP router identifier 8.8.8.8, local AS number 65003
BGP table version is 26, main routing table version 26
6 network entries using 816 bytes of memory
13 path entries using 728 bytes of memory
10/5 BGP path/bestpath attribute entries using 1280 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2920 total bytes of memory
BGP activity 7/1 prefixes, 37/24 paths, scan interval 60 secs
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
7.7.7.7         4        65003       9      11       26    0    0 00:01:47        6
192.168.48.4    4            1     142     133       26    0    0 01:47:10        6
```

从R7 ping R8；
```c
R7(config-router)#do p 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R7(config-router)#do p 8.8.8.8 source 7.7.7.7
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
Packet sent with a source address of 7.7.7.7 
.....
Success rate is 0 percent (0/5)
//R7无法ping通R8；
```

查看R7的路由表；
```c
R7(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.47.4, 01:44:45
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.47.4, 01:31:01
        7.0.0.0/32 is subnetted, 1 subnets
C        7.7.7.7 is directly connected, Loopback0
        8.0.0.0/32 is subnetted, 1 subnets
B        8.8.8.8 [200/0] via 8.8.8.8, 00:01:41
B     192.168.25.0/24 [20/0] via 192.168.47.4, 01:31:51
B     192.168.26.0/24 [20/0] via 192.168.47.4, 01:31:26
        192.168.47.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.47.0/24 is directly connected, FastEthernet0/0
L        192.168.47.7/32 is directly connected, FastEthernet0/0
```

查看R8的路由表；
```c
R8(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.48.4, 01:47:33
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.48.4, 01:34:33
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [200/0] via 7.7.7.7, 00:02:03
        8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback0
B     192.168.25.0/24 [20/0] via 192.168.48.4, 01:35:24
B     192.168.26.0/24 [20/0] via 192.168.48.4, 01:34:59
        192.168.48.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.48.0/24 is directly connected, FastEthernet0/1
L        192.168.48.8/32 is directly connected, FastEthernet0/1
```

查看R7的BGP表；
```c
R7(config-router)#do sh ip bgp
BGP table version is 114, local router ID is 7.7.7.7
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
* i5.5.5.5/32       192.168.48.4             0    100      0 1 65001 i
*>                  192.168.47.4                           0 1 65001 i
* i6.6.6.6/32       192.168.48.4             0    100      0 1 65002 i
*>                  192.168.47.4                           0 1 65002 i
* i7.7.7.7/32       192.168.48.4             0    100      0 1 65003 i//可以看到R7的直连环回口却又通过PE被通告了回来，有环路存在；
*                   192.168.47.4                           0 1 65003 i//可以看到R7的直连环回口却又通过PE被通告了回来，有环路存在；
*>                  0.0.0.0                  0         32768 i
*>i8.8.8.8/32       8.8.8.8                  0    100      0 i
*                   192.168.47.4                           0 1 65003 i
* i192.168.25.0     192.168.48.4             0    100      0 1 ?
*>                  192.168.47.4                           0 1 ?
* i192.168.26.0     192.168.48.4             0    100      0 1 ?
*>                  192.168.47.4                           0 1 ?
```

查看R8的BGP表；
```c
R8(config-router)#do sh ip bgp
BGP table version is 116, local router ID is 8.8.8.8
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
* i5.5.5.5/32       192.168.47.4             0    100      0 1 65001 i
*>                  192.168.48.4                           0 1 65001 i
* i6.6.6.6/32       192.168.47.4             0    100      0 1 65002 i
*>                  192.168.48.4                           0 1 65002 i
*>i7.7.7.7/32       7.7.7.7                  0    100      0 i
*                   192.168.48.4                           0 1 65003 i
* i8.8.8.8/32       192.168.47.4             0    100      0 1 65003 i//可以看到R8的直连环回口却又通过PE被通告了回来，有环路存在；
*                   192.168.48.4                           0 1 65003 i//可以看到R8的直连环回口却又通过PE被通告了回来，有环路存在；
*>                  0.0.0.0                  0         32768 i
* i192.168.25.0     192.168.47.4             0    100      0 1 ?
*>                  192.168.48.4                           0 1 ?
* i192.168.26.0     192.168.47.4             0    100      0 1 ?
*>                  192.168.48.4                           0 1 ?
```
可以看到，现在虽然将R7和R8配置为iBGP邻居，并且由于MPLS VPN的存在R7和R8通过MPLS VPN将iBGP路由发送给对方，而由PE通告的eBGP路由具有更长的AS_PATH，所以R7和R8会优选自己通告的BGP路由。

但是实际上它们之间的连通性是由同PE的eBGP邻居提供的，如果不走PE，就无法到达对方！

**解决方案是，不让R7和R8建立iBGP邻居；**


删除R7的iBGP邻居配置；
```c
R7(config-router)#no nei 8.8.8.8 remote-as 65003
```

删除R8的iBGP邻居配置；
```c
R8(config-router)#no nei 7.7.7.7 remote-as 65003
```

查看R7的路由表；
```c
R7(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.47.4, 03:22:49
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.47.4, 03:09:05
        7.0.0.0/32 is subnetted, 1 subnets
C        7.7.7.7 is directly connected, Loopback0
        8.0.0.0/32 is subnetted, 1 subnets
B        8.8.8.8 [20/0] via 192.168.47.4, 00:00:11
B     192.168.25.0/24 [20/0] via 192.168.47.4, 03:09:55
B     192.168.26.0/24 [20/0] via 192.168.47.4, 03:09:30
        192.168.47.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.47.0/24 is directly connected, FastEthernet0/0
L        192.168.47.7/32 is directly connected, FastEthernet0/0
```

查看R8的路由表；
```c
R8(config-router)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
B        5.5.5.5 [20/0] via 192.168.48.4, 03:24:23
        6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.48.4, 03:11:23
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [20/0] via 192.168.48.4, 00:02:34
        8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback0
B     192.168.25.0/24 [20/0] via 192.168.48.4, 03:12:14
B     192.168.26.0/24 [20/0] via 192.168.48.4, 03:11:49
        192.168.48.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.48.0/24 is directly connected, FastEthernet0/1
L        192.168.48.8/32 is directly connected, FastEthernet0/1
```

查看R7的BGP表；
```c
R7(config)#do sh ip bgp                                                   
BGP table version is 147, local router ID is 7.7.7.7
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
*> 5.5.5.5/32       192.168.47.4                           0 1 65001 i
*> 6.6.6.6/32       192.168.47.4                           0 1 65002 i
*  7.7.7.7/32       192.168.47.4                           0 1 65003 i
*>                  0.0.0.0                  0         32768 i
*> 8.8.8.8/32       192.168.47.4                           0 1 65003 i
*> 192.168.25.0     192.168.47.4                           0 1 ?
*> 192.168.26.0     192.168.47.4                           0 1 ?
//虽然现在把下一跳解决了，但是环路依然存在；
```

查看R8的BGP表；
```c
R8(config-router)#do sh ip bgp 
BGP table version is 147, local router ID is 8.8.8.8
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
*> 5.5.5.5/32       192.168.48.4                           0 1 65001 i
*> 6.6.6.6/32       192.168.48.4                           0 1 65002 i
*> 7.7.7.7/32       192.168.48.4                           0 1 65003 i
*  8.8.8.8/32       192.168.48.4                           0 1 65003 i
*>                  0.0.0.0                  0         32768 i
*> 192.168.25.0     192.168.48.4                           0 1 ?
*> 192.168.26.0     192.168.48.4                           0 1 ?
//虽然现在把下一跳解决了，但是环路依然存在；
```

在R4上配置：
```c
R4(config)#route-map SOO1:1 permit 10    
R4(config-route-map)#set extcommunity soo 1:1      
R4(config-route-map)#exit
R4(config)#route-map SOO1:2 permit 10
R4(config-route-map)#set extcommunity soo 1:2  
R4(config-route-map)#exit

R4(config)#router bgp 1
R4(config-router)#add ipv4 vrf A-Site3
R4(config-router-af)#neighbor 192.168.47.7 route-map SOO1:1 in
R4(config-router-af)#neighbor 192.168.48.8 route-map SOO1:2 in
```

```c
R7#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

R7#ping 8.8.8.8 source l0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
Packet sent with a source address of 7.7.7.7 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 44/64/88 ms
//指定源后，成功ping通！
//发现是之前疏忽大意了，忘了在预配中将R4的直连路由重分发进MP-BGP了；
```

检查配置，R4和R1上将直连路由重分发进MP-BGP；
```c
R4(config)#router bgp 1
R4(config-router)#add ipv4 vrf A-Site3
R4(config-router-af)#redistribute connected

R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
R1(config-router-af)#redistribute connected
```

从R7上ping其他站点的路由器；
```c
R7#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 36/84/120 ms
R7#ping 5.5.5.5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 5.5.5.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 88/121/144 ms
R7#ping 6.6.6.6  
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 6.6.6.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 80/105/140 ms
```

查看R5的路由表，发现R5依然没有R6的路由；
```c
R5#sh ip route 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        5.0.0.0/32 is subnetted, 1 subnets
C        5.5.5.5 is directly connected, Loopback0
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [20/0] via 192.168.15.1, 04:17:22
        8.0.0.0/32 is subnetted, 1 subnets
B        8.8.8.8 [20/0] via 192.168.15.1, 04:16:37
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.5/32 is directly connected, FastEthernet0/1
        192.168.25.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.25.0/24 is directly connected, FastEthernet0/0
L        192.168.25.5/32 is directly connected, FastEthernet0/0
B     192.168.47.0/24 [20/0] via 192.168.15.1, 00:11:01
B     192.168.48.0/24 [20/0] via 192.168.15.1, 00:11:01
```

查看R6的路由表，发现R6同样没有R5的路由；
```c
R6#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        6.0.0.0/32 is subnetted, 1 subnets
C        6.6.6.6 is directly connected, Loopback0
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [20/0] via 192.168.26.2, 04:04:46
        8.0.0.0/32 is subnetted, 1 subnets
B        8.8.8.8 [20/0] via 192.168.26.2, 04:04:46
        192.168.26.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.26.0/24 is directly connected, FastEthernet0/0
L        192.168.26.6/32 is directly connected, FastEthernet0/0
B     192.168.47.0/24 [20/0] via 192.168.26.2, 00:12:07
B     192.168.48.0/24 [20/0] via 192.168.26.2, 00:12:07
```

由于需求要求Spoke和Spoke间的通信需要先绕经Hub站点，所以R5需要从站点3学到关于R6的路由，R6也需要从站点3学到关于R5的路由；但当路由从R4通告给站点3的时候，路由的AS_PATH携带了AS1，当被站点3通告回R4的时候，R4就会丢弃AS_PATH中携带了和自己所在AS号相同的的路由。
            
        
        
# 总结：
PE路由器简单地比较CE路由器ASN和 `as-path` 中的ASN，如果匹配的话，所有在 `as-path` 中的该ASN都会被服务提供商的ASN所代替。
这样一来，远程CE就会接受这些路由了，因为它们在这些BGP路由的 `as-path` 中看不到自己的ASN了！但这样一来，对于可能存在环路的保护机制，以及非最优路由的 `as-path` 检查机制就失效了。因此，在使用 `as-override` 功能进行ASN覆盖的时候，明智的做法是为BGP实施SOO特性。
