---
title: MPLS 实验9：OSPF Sham Link and Backdoor Link in MPLS VPN
date: 2014-03-19 20:28:51
updated: 2014-03-19 20:28:51
tags: [MPLS, Network]
categories: Network
---

# 实验环境：
- 模拟器：GNS3 0.8.6
- Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image

# 实验拓扑：
![](topo.png)

<!-- more -->

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
    route-target import 1:1
!
mpls label protocol ldp
!
interface Loopback0
    ip address 1.1.1.1 255.255.255.255
    ip ospf 1 area 0
!
interface Loopback1
    ip vrf forwarding A-Site1
    ip address 11.11.11.11 255.255.255.255
!
interface FastEthernet0/0
    ip address 192.168.12.1 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip  
!
interface FastEthernet0/1
    ip vrf forwarding A-Site1
    ip address 192.168.13.1 255.255.255.0
    ip ospf 2 area 0
    no shutdown
!
router ospf 2 vrf A-Site1
    router-id 11.11.11.11
    redistribute bgp 1 subnets
!
router ospf 1
    router-id 1.1.1.1
!
router bgp 1
    bgp log-neighbor-changes
    neighbor 2.2.2.2 remote-as 1
    neighbor 2.2.2.2 update-source Loopback0
    neighbor 2.2.2.2 next-hop-self
    !
    address-family vpnv4
    neighbor 2.2.2.2 activate
    neighbor 2.2.2.2 send-community both
    exit-address-family
    !
    address-family ipv4 vrf A-Site1
    redistribute connected
    redistribute ospf 2 match internal external 1 external 2
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
ip vrf A-Site2
    rd 1:2
    route-target export 1:1
    route-target import 1:1
!
mpls label protocol ldp
!
interface Loopback0
    ip address 2.2.2.2 255.255.255.255
    ip ospf 1 area 0
!
interface Loopback1
    ip vrf forwarding A-Site2
    ip address 22.22.22.22 255.255.255.255
!
interface FastEthernet0/0
    ip address 192.168.12.2 255.255.255.0
    ip ospf 1 area 0
    no shutdown
mpls ip  
!
interface FastEthernet0/1
    ip vrf forwarding A-Site2
    ip address 192.168.24.2 255.255.255.0
    ip ospf 2 area 0
    no shutdown
!
router ospf 2 vrf A-Site2
    router-id 22.22.22.22
    redistribute bgp 1 subnets
!
router ospf 1
    router-id 2.2.2.2
!
router bgp 1
    bgp log-neighbor-changes
    neighbor 1.1.1.1 remote-as 1
    neighbor 1.1.1.1 update-source Loopback0
    neighbor 1.1.1.1 next-hop-self
    !
    address-family vpnv4
    neighbor 1.1.1.1 activate
    neighbor 1.1.1.1 send-community both
    exit-address-family
    !
    address-family ipv4 vrf A-Site2
    redistribute connected
    redistribute ospf 2 match internal external 1 external 2
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
interface Loopback0
    ip address 3.3.3.3 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/1
    ip address 192.168.13.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface Serial1/0
    ip address 192.168.34.3 255.255.255.0
    ip ospf 1 area 0
    no shutdown
    clock rate 64000
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
    
## R4:
```c
hostname R4
!
ip cef
!
interface Loopback0
    ip address 4.4.4.4 255.255.255.255
    ip ospf 1 area 0
!
interface FastEthernet0/1
    ip address 192.168.24.4 255.255.255.0
    ip ospf 1 area 0
    no shutdown
!
interface Serial1/0
    ip address 192.168.34.4 255.255.255.0
    ip ospf 1 area 0
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
        
        
# 实验与调试：
## 实验1：当在VPN站点之间存在后门备份链路时，使用OSPF Sham Link解决后门链路优先于MPLS VPN链路的问题；

查看R3的路由表；
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
        3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/65] via 192.168.34.4, 00:03:07, Serial1/0
        192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/24 is directly connected, FastEthernet0/1
L        192.168.13.3/32 is directly connected, FastEthernet0/1
O     192.168.24.0/24 [110/65] via 192.168.34.4, 00:03:07, Serial1/0
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, Serial1/0
L        192.168.34.3/32 is directly connected, Serial1/0
//此时，R3去往4.4.4.4/32和192.168.24.0/24是通过走后门链路；
//而后门链路通常是作为备份链路，在主链路正常的情况下，通常不希望，走备份链路；
```
查看R3的OSPF Database；
```c
R3(config-if)#do sh ip  ospf 1 database
            OSPF Router with ID (3.3.3.3) (Process ID 1)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
3.3.3.3         3.3.3.3         10          0x80000006 0x003904 4
4.4.4.4         4.4.4.4         11          0x80000004 0x0049D4 4
11.11.11.11     11.11.11.11     110         0x80000001 0x00B333 1
22.22.22.22     22.22.22.22     95          0x80000002 0x0075FF 1
192.168.24.4    192.168.24.4    1384        0x80000005 0x00DADF 4
                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.13.3    3.3.3.3         108         0x80000001 0x007408
192.168.24.4    4.4.4.4         94          0x80000001 0x001D1F
                Summary Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
3.3.3.3         22.22.22.22     99          0x80000001 0x00F461
4.4.4.4         11.11.11.11     89          0x80000001 0x00126C
192.168.13.0    22.22.22.22     99          0x80000001 0x0034B8
192.168.24.0    11.11.11.11     99          0x80000001 0x000608
                Type-5 AS External Link States
Link ID         ADV Router      Age         Seq#       Checksum Tag
11.11.11.11     22.22.22.22     99          0x80000001 0x00E478 3489660929
22.22.22.22     11.11.11.11     109         0x80000001 0x003429 3489660929
```
注意到，从MPLS VPN过来的LSA是类型3的LSA，而从后门链路过来的LSA是类型1的LSA；以前的实验说过，在OSPF看来，MPLS VPN超骨干网络超越了OSPF骨干区域Area0，MPLS VPN超骨干网络神似OSPF骨干区域Area0，但它又和OSPF骨干区域Area0不同，所以PE路由器会执行ABR的功能，会将类型为1和2的LSA转换为类型3的LSA；所以当PE在收到来自对端远程PE发来的OSPF区域内路由（O）时，会将OSPF区域内路由转换为OSPF区域间路由（O IA）。

为了避免PE将类型1和类型2的LSA转换为类型3的LSA，可以在PE之间创建OSPF Sham Link；这样，当LSA在Sham Link中进行洪泛时，所有的OSPF路由类型都不会改变，不会转变为类型3或者类型5的LSA；


**在PE上配置OSPF Sham Link；**

```c
注意：必须将Sham Link的源地址和目的地址所在的接口划分进VRF；
R1(config)#router ospf 2 vrf A-Site1
R1(config-router)#area 0 sham-link 11.11.11.11 22.22.22.22

R2(config)#router ospf 2 vrf A-Site2
R2(config-router)#area 0 sham-link 22.22.22.22 11.11.11.11

//将接口Loopback1的地址以路由的形式通告进MP-BGP的IPv4 VRF地址族；
//注意：必须在PE上将各自用作Sham Link的源地址的接口地址通告进MP-BGP的IPv4 VRF地址族，否则Sham Link会反复震荡；
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
R1(config-router-af)#network 11.11.11.11 mask 255.255.255.255

R2(config)#router bgp 1
R2(config-router)#address-family ipv4 vrf A-Site2
R2(config-router-af)#network 22.22.22.22 mask 255.255.255.255
```
            
将作为Sham Link源地址，并且同时也作为了对方PE Sham Link目的地址的接口移除OSPF VRF进程；
> 备注：虽然在此次实验中，我没有将R1和R2的Loopback1通告进OSPF VRF进程，但仍需注意这一点；

> 注意：确保作为Sham Link源地址和目的地址的接口不能被通告进OSPF VRF进程；
```c
R1(config)#router ospf 2 vrf A-Site1               
R1(config-router)#no network 11.11.11.11 0.0.0.0 area 0

R2(config)#router ospf 2 vrf A-Site2
R2(config-router)#no network 22.22.22.22 0.0.0.0 area 0
```
            
查看R3的路由表；
```c
R3(config-if)#do sh ip route           
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/4] via 192.168.13.1, 00:01:09, FastEthernet0/1 //注意路由的度量值；
        11.0.0.0/32 is subnetted, 1 subnets
O E2     11.11.11.11 [110/1] via 192.168.13.1, 00:03:04, FastEthernet0/1
        22.0.0.0/32 is subnetted, 1 subnets
O E2     22.22.22.22 [110/1] via 192.168.13.1, 00:15:51, FastEthernet0/1
        192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/24 is directly connected, FastEthernet0/1
L        192.168.13.3/32 is directly connected, FastEthernet0/1
O     192.168.24.0/24 [110/3] via 192.168.13.1, 00:01:09, FastEthernet0/1
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, Serial1/0
L        192.168.34.3/32 is directly connected, Serial1/0
```

查看R4的路由表；
```c
R4(config-if)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/4] via 192.168.24.2, 00:02:12, FastEthernet0/1 //注意路由的度量值；
        4.0.0.0/32 is subnetted, 1 subnets
C        4.4.4.4 is directly connected, Loopback0
        11.0.0.0/32 is subnetted, 1 subnets
O E2     11.11.11.11 [110/1] via 192.168.24.2, 00:16:39, FastEthernet0/1
        22.0.0.0/32 is subnetted, 1 subnets
O E2     22.22.22.22 [110/1] via 192.168.24.2, 00:02:12, FastEthernet0/1
O     192.168.13.0/24 [110/3] via 192.168.24.2, 00:02:12, FastEthernet0/1
        192.168.24.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.24.0/24 is directly connected, FastEthernet0/1
L        192.168.24.4/32 is directly connected, FastEthernet0/1
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, Serial1/0
L        192.168.34.4/32 is directly connected, Serial1/0
//可以看到R3和R4现在都通过MPLS VPN去往对方所在站点；
```

在必要时还可以修改Sham Link的开销，让Sham Link比后门链路的度量值更低，以便优先选择作为主链路的MPLS VPN，而不是备份的后门链路；
```c
R1(config)#router ospf 2 vrf A-Site1
R1(config-router)# area 0 sham-link 11.11.11.11 22.22.22.22 cost 10

R2(config-router-af)#router ospf 2 vrf A-Site2
R2(config-router)#area 0 sham-link 22.22.22.22 11.11.11.11 cost 10
```

查看R3的路由表；
```c
R3(config-if)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/13] via 192.168.13.1, 00:00:58, FastEthernet0/1 //注意路由的度量值变化；
        11.0.0.0/32 is subnetted, 1 subnets
O E2     11.11.11.11 [110/1] via 192.168.13.1, 00:04:59, FastEthernet0/1
        22.0.0.0/32 is subnetted, 1 subnets
O E2     22.22.22.22 [110/1] via 192.168.13.1, 00:17:46, FastEthernet0/1
        192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/24 is directly connected, FastEthernet0/1
L        192.168.13.3/32 is directly connected, FastEthernet0/1
O     192.168.24.0/24 [110/12] via 192.168.13.1, 00:00:58, FastEthernet0/1
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, Serial1/0
L        192.168.34.3/32 is directly connected, Serial1/0
```
                
查看R4的路由表；
```c
R4(config-if)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
        D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
        N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
        E1 - OSPF external type 1, E2 - OSPF external type 2
        i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
        ia - IS-IS inter area, * - candidate default, U - per-user static route
        o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
        + - replicated route, % - next hop override
Gateway of last resort is not set
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/13] via 192.168.24.2, 00:02:11, FastEthernet0/1 //注意路由的度量值变化；
        4.0.0.0/32 is subnetted, 1 subnets
C        4.4.4.4 is directly connected, Loopback0
        11.0.0.0/32 is subnetted, 1 subnets
O E2     11.11.11.11 [110/1] via 192.168.24.2, 00:19:40, FastEthernet0/1
        22.0.0.0/32 is subnetted, 1 subnets
O E2     22.22.22.22 [110/1] via 192.168.24.2, 00:05:13, FastEthernet0/1
O     192.168.13.0/24 [110/12] via 192.168.24.2, 00:02:11, FastEthernet0/1
        192.168.24.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.24.0/24 is directly connected, FastEthernet0/1
L        192.168.24.4/32 is directly connected, FastEthernet0/1
        192.168.34.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.34.0/24 is directly connected, Serial1/0
L        192.168.34.4/32 is directly connected, Serial1/0
```