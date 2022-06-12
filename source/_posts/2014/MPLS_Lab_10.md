---
title: MPLS 实验10：OSPF Loop Prevention in MPLS VPN
date: 2014-03-19 20:29:51
updated: 2014-03-19 20:29:51
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

# 基本预配置：
## R1：
```c
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
        no shutdown
!
interface FastEthernet0/1
        ip address 192.168.13.1 255.255.255.0
        ip ospf 1 area 0
        no shutdown
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

## R2:
```c
hostname R2
!
ip cef
!
ip vrf A-Site1
        rd 1:1
//注意：连接到同一站点的同一ISP的多个不同PE可以使用相同的RD，也可以使用不同的RD；
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
        ip vrf forwarding A-Site1
        ip address 22.22.22.22 255.255.255.255
        ip ospf 2 area 0
!
interface FastEthernet0/0
        ip vrf forwarding A-Site1
        ip address 192.168.12.2 255.255.255.0
        ip ospf 2 area 0
        no shutdown
!
interface FastEthernet0/1
        ip address 192.168.24.2 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
router ospf 2 vrf A-Site1
        router-id 22.22.22.22
        redistribute bgp 1 subnets
!
router ospf 1
        router-id 2.2.2.2
!
router bgp 1
        bgp log-neighbor-changes
        neighbor 4.4.4.4 remote-as 1
        neighbor 4.4.4.4 update-source Loopback0
        neighbor 4.4.4.4 next-hop-self
        !
        address-family vpnv4
        neighbor 4.4.4.4 activate
        neighbor 4.4.4.4 send-community both
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
	
## R3:
```c
hostname R3
!
ip cef
!
ip vrf A-Site1
        rd 1:1
//注意：连接到同一站点的同一ISP的多个不同PE可以使用相同的RD，也可以使用不同的RD；
        route-target export 1:1
        route-target import 1:1
!
mpls label protocol ldp
!
interface Loopback0
        ip address 3.3.3.3 255.255.255.255
        ip ospf 1 area 0
!
interface Loopback1
        ip vrf forwarding A-Site1
        ip address 33.33.33.33 255.255.255.255
        ip ospf 2 area 0
!
interface FastEthernet0/0
        ip address 192.168.34.3 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet0/1
        ip vrf forwarding A-Site1
        ip address 192.168.13.3 255.255.255.0
        ip ospf 2 area 0
        no shutdown
!
router ospf 2 vrf A-Site1
        router-id 33.33.33.33
        redistribute bgp 1 subnets
!
router ospf 1
        router-id 3.3.3.3
!
router bgp 1
        bgp log-neighbor-changes
        neighbor 4.4.4.4 remote-as 1
        neighbor 4.4.4.4 update-source Loopback0
        neighbor 4.4.4.4 next-hop-self
        !
        address-family vpnv4
        neighbor 4.4.4.4 activate
        neighbor 4.4.4.4 send-community both
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
	
## R4:
```c
hostname R4
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
        ip address 4.4.4.4 255.255.255.255
        ip ospf 1 area 0
!
interface Loopback1
        ip vrf forwarding A-Site2
        ip address 44.44.44.44 255.255.255.255
        ip ospf 2 area 0
!
interface FastEthernet0/0
        ip address 192.168.34.4 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet0/1
        ip address 192.168.24.4 255.255.255.0
        ip ospf 1 area 0
        no shutdown
mpls ip
!
interface FastEthernet1/0
        ip vrf forwarding A-Site2
        ip address 192.168.45.4 255.255.255.0
        ip ospf 2 area 0
        no shutdown
!
router ospf 2 vrf A-Site2
        router-id 44.44.44.44
        redistribute bgp 1 subnets
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
        neighbor 2.2.2.2 peer-group ibgp
        neighbor 3.3.3.3 peer-group ibgp
        !
        address-family vpnv4
        neighbor ibgp send-community both
        neighbor 2.2.2.2 activate
        neighbor 3.3.3.3 activate
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
	
## R5:
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
        ip address 192.168.45.5 255.255.255.0
        ip ospf 1 area 0
        no shutdown
!
router ospf 1
        router-id 5.5.5.5
        redistribute connected subnets route-map CREATE_LSA5
//为了创造出类型5的LSA；
!
route-map CREATE_LSA5 permit 10
        match interface Loopback1
//为了创造出类型5的LSA；
!
line con 0
        exec-timeout 0 0
        logging synchronous
!
end
```
		
		
# 实验与调试：
## 实验1：OSPF内部路由Down-bit
