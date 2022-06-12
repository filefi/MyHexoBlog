---
title: MPLS 实验12：SOO and EIGRP on Backdoor Link
date: 2014-03-19 23:49:51
updated: 2014-03-19 23:49:51
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
interface FastEthernet0/0
        ip address 192.168.12.1 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet0/1
        ip vrf forwarding A-Site1
        ip address 192.168.13.1 255.255.255.0
        no shutdown
!
!
router eigrp 1
        !
        address-family ipv4 vrf A-Site1 autonomous-system 1
        redistribute bgp 1 metric 10000 100 255 1 1500
        network 192.168.13.0
        eigrp router-id 1.1.1.1
        exit-address-family
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
        network 11.11.11.11 mask 255.255.255.255
        redistribute connected
        redistribute eigrp 1
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
interface FastEthernet0/0
        ip address 192.168.12.2 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet0/1
        ip vrf forwarding A-Site2
        ip address 192.168.24.2 255.255.255.0
        no shutdown
!
router eigrp 1
        !
        address-family ipv4 vrf A-Site2 autonomous-system 1
        redistribute bgp 1 metric 10000 100 255 1 1500
        network 192.168.24.0
        eigrp router-id 2.2.2.2
        exit-address-family
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
        network 22.22.22.22 mask 255.255.255.255
        redistribute connected
        redistribute eigrp 1
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
!
interface FastEthernet0/1
        ip address 192.168.13.3 255.255.255.0
        no shutdown
!
interface Serial1/0
        ip address 192.168.34.3 255.255.255.0
        serial restart-delay 0
        clock rate 64000
        no shutdown
!
router eigrp 1
        network 3.3.3.3 0.0.0.0
        network 192.168.13.0
        network 192.168.34.0
        eigrp router-id 3.3.3.3
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
!
interface FastEthernet0/1
        ip address 192.168.24.4 255.255.255.0
        no shutdown
!
interface Serial1/0
        ip address 192.168.34.4 255.255.255.0
        serial restart-delay 0
        no shutdown
!
router eigrp 1
        network 4.4.4.4 0.0.0.0
        network 192.168.24.0
        network 192.168.34.0
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
        


# 实验与调试：
## 实验1：
查看R1的IP BGP VNPv4路由表关于路由4.4.4.4的明细；
```c
R1(config-if)#do sh ip bgp vpnv4 all 4.4.4.4
BGP routing table entry for 1:1:4.4.4.4/32, version 79
Paths: (1 available, best #1, table A-Site1)
Not advertised to any peer
Local, imported path from 1:2:4.4.4.4/32
2.2.2.2 (metric 2) from 2.2.2.2 (2.2.2.2)
        Origin incomplete, metric 156160, localpref 100, valid, internal, best
        Extended Community: RT:1:1 Cost:pre-bestpath:128:156160 0x8800:32768:0 
        0x8801:1:130560 0x8802:65281:25600 0x8803:65281:1500 0x8806:0:67372036
        mpls labels in/out nolabel/22
BGP routing table entry for 1:2:4.4.4.4/32, version 77
Paths: (1 available, best #1, no table)
Not advertised to any peer
Local
2.2.2.2 (metric 2) from 2.2.2.2 (2.2.2.2)
        Origin incomplete, metric 156160, localpref 100, valid, internal, best
        Extended Community: RT:1:1 Cost:pre-bestpath:128:156160 0x8800:32768:0 
        0x8801:1:130560 0x8802:65281:25600 0x8803:65281:1500 0x8806:0:67372036
        mpls labels in/out nolabel/22
//VPNv4路由默认不携带SOO；
```

在PE上，创建路由映射表SOO_LAB，对所有VPNv4路由设置MP-BGP拓展团体属性；
```c
R1(config)#route-map SOO_LAB permit 10
R1(config-route-map)#set extcommunity ?
cost  Cost extended community
rt    Route Target extended community
soo   Site-of-Origin extended community

R1(config-route-map)#set extcommunity soo ?
ASN:nn or IP-address:nn  VPN extended community

R1(config-route-map)#set extcommunity soo 1:1
R1(config-route-map)#exit

R2(config)#route-map SOO_LAB permit 10 
R2(config-route-map)#set extcommunity soo 1:2
R2(config-route-map)#exit
```
            
在PE的vrf接口上应用设置SOO的`route-map`；
```c
R1(config)#int f0/1
R1(config-if)#ip vrf sitemap ?
WORD  Name of the route-map
R1(config-if)#ip vrf sitemap SOO_LAB


R2(config)#int f0/1
R2(config-if)#ip vrf sitemap SOO_LAB
```
            
            
在连接到后门链路的CE或C路由器上为EIGRP配置SOO；
```c
R3(config)#route-map SOO_LAB permit 10 
R3(config-route-map)#set extcommunity soo 1:1

R3(config)#int f0/1
R3(config-if)#ip vrf sitemap SOO_LAB

R4(config)#route-map SOO_LAB permit 10 
R4(config-route-map)#set extcommunity soo 1:2

R4(config)#int f0/1
R4(config-if)#ip vrf sitemap SOO_LAB
```
        
查看R1的IP BGP VNPv4路由表关于路由4.4.4.4的明细；
```c
R2(config-if)#do sh ip bgp vpnv4 all 3.3.3.3
BGP routing table entry for 1:1:3.3.3.3/32, version 96
Paths: (1 available, best #1, no table)
Not advertised to any peer
Local
1.1.1.1 (metric 2) from 1.1.1.1 (1.1.1.1)
        Origin incomplete, metric 156160, localpref 100, valid, internal, best
        Extended Community: SoO:1:1 RT:1:1 Cost:pre-bestpath:128:156160 
        0x8800:32768:0 0x8801:1:130560 0x8802:65281:25600 0x8803:65281:1500 
        0x8806:0:50529027
        mpls labels in/out nolabel/24
BGP routing table entry for 1:2:3.3.3.3/32, version 99
Paths: (1 available, best #1, table A-Site2)
Not advertised to any peer
Local, imported path from 1:1:3.3.3.3/32
1.1.1.1 (metric 2) from 1.1.1.1 (1.1.1.1)
        Origin incomplete, metric 156160, localpref 100, valid, internal, best
        Extended Community: SoO:1:1 RT:1:1 Cost:pre-bestpath:128:156160 
        0x8800:32768:0 0x8801:1:130560 0x8802:65281:25600 0x8803:65281:1500 
        0x8806:0:50529027
        mpls labels in/out nolabel/24
```

查看R3的EIGRP 拓扑表关于4.4.4.4/32的明细；
```c
R3#sh ip ei topology 4.4.4.4/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(3.3.3.3) for 4.4.4.4/32
State is Passive, Query origin flag is 1, 1 Successor(s), FD is 158720
Descriptor Blocks:
192.168.13.1 (FastEthernet0/1), from 192.168.13.1, Send flag is 0x0
        Composite metric is (158720/156160), route is Internal
        Vector metric:
        Minimum bandwidth is 100000 Kbit
        Total delay is 5200 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 4.4.4.4
        Extended Community: SoO:1:2
192.168.34.4 (Serial1/0), from 192.168.34.4, Send flag is 0x0
        Composite metric is (2297856/128256), route is Internal
        Vector metric:
        Minimum bandwidth is 1544 Kbit
        Total delay is 25000 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 1
        Originating router is 4.4.4.4
//R3上来自于R4的关于4.4.4.4/32的路由携带了SOO1:2；
```

查看R4的EIGRP 拓扑表关于3.3.3.3/32的明细；
```c
R4(config-if)#do sh ip ei to 3.3.3.3/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(4.4.4.4) for 3.3.3.3/32
State is Passive, Query origin flag is 1, 1 Successor(s), FD is 158720
Descriptor Blocks:
192.168.24.2 (FastEthernet0/1), from 192.168.24.2, Send flag is 0x0
        Composite metric is (158720/156160), route is Internal
        Vector metric:
        Minimum bandwidth is 100000 Kbit
        Total delay is 5200 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 3.3.3.3
        Extended Community: SoO:1:1
192.168.34.3 (Serial1/0), from 192.168.34.3, Send flag is 0x0
        Composite metric is (2297856/128256), route is Internal
        Vector metric:
        Minimum bandwidth is 1544 Kbit
        Total delay is 25000 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 1
        Originating router is 3.3.3.3
        //R4上来自于R3的关于3.3.3.3/32的路由携带了SOO1:1；
```

