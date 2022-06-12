---
title: MPLS 实验8：MPLS VPN running OSPF on the PE-CE link
date: 2014-03-14 22:58:51
updated: 2014-03-14 22:58:51
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
```c
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
```c
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


# MPLS VPN配置步骤：
1. 创建VRF（VRF名本地有效）：
   - 指定RD（提供全局唯一的私网单播地址）；
   - 指定RT的导出（导出：把重分发进MP-BGP的VPNv4路由打上MP-BGP扩展团体属性RT）和导入（导入：把MP-BGP里的VPNv4路由进行RT的匹配，匹配成功的VPNv4路由将放进相应的VRF）；
   - 将与CE相连的PE接口关联特定VRF；
2. 配置MP-BGP：
   - 配置建立PE之间的IBGP邻居关系；
   - 启用VPNv4地址族（AF），并激活与其他PE设备的邻居关系；
3. 配置PE-CE路由；
   - 配置IGP，并启用IGP的地址族（AF）；
   - 配置启用MP-BGP IPv4 VRF地址族，然后激活与其他PE路由器的MP-BGP IPv4 VRF邻居关系；
    
    
# 实验与调试：
## 实验1：配置MPLS VPN
### 在PE上创建VRF；
在PE1（R1）上创建VRF，并命名为A-Site1，表示该VRF为VPN A的站点1服务；
```c
R1(config)#ip vrf A-Site1   
```

指定RD，为1:1，如果按照AS：num命名法，以下RD表示AS1中的第2个VRF；
```c
R1(config-vrf)#rd 1:1
```

指定RT的导入和导出，这里我将RT指定为导入和导出1：1；
```c
R1(config-vrf)#route-target ?        
    ASN:nn or IP-address:nn  Target VPN Extended Community
    both                     Both import and export Target-VPN community
    export                   Export Target-VPN community
    import                   Import Target-VPN community
R1(config-vrf)#route-target both 1:1
```

将PE1（R1）上与CE相连的PE接口F0/1关联到VRF A-Site1；
```c
R1(config)#int f0/1
R1(config-if)#ip vrf forwarding A-Site1
% Interface FastEthernet0/1 IPv4 disabled and address(es) removed due to enabling VRF A-Site1
    //切记先将接口关联VRF，然后再配置IP地址；
R1(config-if)#ip add 192.168.15.1 255.255.255.0
```
                
                
在PE2（R4）上创建VRF，并命名为A-Site2，表示该VRF为VPN A的站点2服务；
```c
R4(config)#ip vrf A-Site2
```

指定RD，为1:2，如果按照AS：num命名法，以下RD表示AS1中的第2个VRF；
```c
R4(config-vrf)#rd 1:2
```

指定RT的导入和导出，这里我将RT指定为导入和导出1：1；
```c
R4(config-vrf)#route-target both 1:1
```
因为PE1导出的RT为1:1，PE2导入的RT正是PE1的导出RT1:1，所以PE1（R1）的RT导出和PE2（R4）的RT导入能够匹配成功，R4能够学到R1导入到MP-BGP中的VPNv4路由；又因为PE2导出的RT为1:1，PE1导入的RT也正好是PE2的导出RT1:1；所以PE2（R4）的RT导出和PE1（R1）的RT导入能够匹配成对，R1能够学到R4导入到MP-BGP中的VPNv4路由

将PE2（R4）上与CE相连的PE接口F0/1关联到VRF A-Site2；
```c
R4(config)#int f0/1
R4(config-if)#ip vrf forwarding A-Site2
% Interface FastEthernet0/1 IPv4 disabled and address(es) removed due to enabling VRF A-Site2
    //切记先将接口关联VRF，然后再配置IP地址；
R4(config-if)#ip add 192.168.47.4 255.255.255.0
```
            
    
### 配置MP-BGP；
在PE1（R1）上配置MP-BGP，使之与其他PE建立IBGP关系，在此实验中只有PE1和PE2两台PE设备；
```c
R1(config)#router bgp 1
R1(config-router)#no auto-summary
R1(config-router)#no syn
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#neighbor 4.4.4.4 remote-as 1
R1(config-router)#neighbor 4.4.4.4 update-source l0
R1(config-router)#neighbor 4.4.4.4 next-hop-self
```

启用PE1（R1）的VPNv4地址族，并激活与IBGP邻居PE2（R4）的MP-BGP VPNv4邻居关系；
```c
R1(config-router)#address-family vpnv4
R1(config-router-af)#neighbor 4.4.4.4 activate
//激活与IBGP邻居R4的VPNv4关系；
R1(config-router-af)#neighbor 4.4.4.4 send-community ?
    both      Send Standard and Extended Community attributes
    extended  Send Extended Community attribute
    standard  Send Standard Community attribute
    <cr>

R1(config-router-af)#neighbor 4.4.4.4 send-community both
//由于BGP团体属性为可选传递属性，所以必须手动指定R1向IBGP邻居R4发送拓展团体属性和标准团体属性；

R1(config-router-af)#exit-address-family
//退出AF配置模式；
```
                
在PE2（R4）上配置MP-BGP，使之与其他PE建立IBGP关系，
```c
R4(config-if)#router bgp 1
R4(config-router)#no auto-summary
R4(config-router)#no syn
R4(config-router)#bgp router-id 4.4.4.4
R4(config-router)#neighbor 1.1.1.1 remote-as 1
R4(config-router)#neighbor 1.1.1.1 update-source l0
R4(config-router)#neighbor 1.1.1.1 next-hop-self
```

启用PE2（R4）的VPNv4地址族，并激活与IBGP邻居PE1（R1）的MP-BGP VPNv4邻居关系；
```c
R4(config-router)#address-family vpnv4
R4(config-router-af)#neighbor 1.1.1.1 activate
R4(config-router-af)#neighbor 1.1.1.1 send-community both
```
                
### 配置PE-CE路由；
配置PE1（R1）的IGP OSPF VRF进程；
```c
R1(config)#interface loopback1
R1(config-if)#ip vrf forwarding A-Site1
R1(config-if)#ip add 11.11.11.11 255.255.255.255
//创建接口Loopback1，以使用此环回口作为OSPF 2 VRF A-Site1进程的Router-id；

R1(config)#router ospf 2 vrf A-Site1
R1(config-router)#router-id 11.11.11.11
R1(config-router)#network 11.11.11.11 0.0.0.0 area 0
//OSPF VRF进程中的命令和常规OSPF进程中的相同，使用network命令将接口Loopback1宣告进OSPF VRF进程；

R1(config-router)#redistribute bgp 1 subnets 
//将MP-BGP中IPv4 VRF地址族的命名与OSPF VRF进程的VRF命名相同的MP-BGP路由重分布进此OSPF VRF进程；
//OSPF VRF进程和MP-BGP中IPv4 VRF地址族（AF）实际上是执行相互重分发的操作；

R1(config)#int f0/1
R1(config-if)#ip ospf 2 area 0
//同时，与常规OSPF进程相同，也可以在接口下将接口宣告进OSPF VRF进程；
```

配置PE1（R1）的MP-BGP IPv4 VRF地址族（AF）；
```c
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
//进入MP-BGP的IPv4 VRF地址族（AF）；

R1(config-router-af)#redistribute ospf 2 vrf A-Site1 match internal external 
//默认情况下，把OSPF重分发进BGP，只会将OSPF内部路由重分发进BGP，
//如果需要将所有OSPF路由（内部和外部路由）重分发进去BGP，需要使用选项match；

R1(config-router-af)#redistribute connected 
//如果用户在CE路由器上ping远程网络的另一个VPN站点中的CE或C路由器，
//为了使其在没有指定其源地址情况下（即默认使用CE路由器出站接口IP地址），Echo Reply包能够有路由并正常返回,
//将PE路由器的直连路由重分布进MP-BGP的IPv4 VRF AF中；

R1(config-router-af)#exit-address-family 
//退出MP-BGP的IPv4 VRF地址族（AF）模式；
```
            
            
配置PE2（R4）的IGP OSPF VRF进程；
```c
R4(config)#interface loopback1
R4(config-if)#ip vrf forwarding A-Site2
R4(config-if)#ip add 11.11.11.11 255.255.255.255

R4(config)#router ospf 2 vrf A-Site2
R4(config-router)#router-id 11.11.11.11
R4(config-router)#network 11.11.11.11 0.0.0.0 area 0

R4(config-router)#redistribute bgp 1 subnets 

R4(config)#int f0/1
R4(config-if)#ip ospf 2 area 0
```

配置PE2（R4）的MP-BGP IPv4 VRF地址族（AF）；
```c
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#redistribute ospf 2 vrf A-Site2 match internal external 
R4(config-router-af)#redistribute connected 
R4(config-router-af)#exit-address-family 
```
            
            
### 验证与调试；

在VPN A站点1的C路由器R6上ping远程VPN A站点2的C路由器R8；
```c
R6#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 140/173/212 ms
//成功！
```

查看PE2（R4）的OSPF链路状态数据库；
```c
R4#show ip ospf database
            OSPF Router with ID (4.4.4.4) (Process ID 1)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1920        0x80000005 0x0048DB 2
2.2.2.2         2.2.2.2         1933        0x80000006 0x0039C6 3
3.3.3.3         3.3.3.3         1978        0x80000006 0x009D26 3
4.4.4.4         4.4.4.4         45          0x80000006 0x003A93 2
                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.12.2    2.2.2.2         1933        0x80000004 0x008922
192.168.23.3    3.3.3.3         1978        0x80000004 0x003C57
192.168.34.4    4.4.4.4         45          0x80000005 0x00EC8D
            OSPF Router with ID (44.44.44.44) (Process ID 2)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
7.7.7.7         7.7.7.7         1944        0x80000007 0x0030CA 3
8.8.8.8         8.8.8.8         1146        0x80000005 0x002E11 2
44.44.44.44     44.44.44.44     1858        0x80000003 0x004D81 2
                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.47.7    7.7.7.7         1944        0x80000002 0x005B55
192.168.78.7    7.7.7.7         1167        0x80000004 0x00F12E
                Summary Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
5.5.5.5         44.44.44.44     1858        0x80000002 0x00FFF4
6.6.6.6         44.44.44.44     1858        0x80000002 0x00DB14
11.11.11.11     44.44.44.44     1858        0x80000002 0x00E0FC
192.168.15.0    44.44.44.44     1858        0x80000002 0x00850C
192.168.56.0    44.44.44.44     1858        0x80000002 0x00CA9C
//可以看到，当关于VPN A站点2的前缀在PE2（R4）上的时候，它们都还是类型1或者类型2的LSA；
//而关于这些前缀的LSA在PE1（R1）的时候，它们
```

查看PE1（R1）的OSPF链路状态数据库；
```c
R1#show ip ospf database
            OSPF Router with ID (1.1.1.1) (Process ID 1)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1406        0x80000005 0x0048DB 2
2.2.2.2         2.2.2.2         1422        0x80000006 0x0039C6 3
3.3.3.3         3.3.3.3         1468        0x80000006 0x009D26 3
4.4.4.4         4.4.4.4         1559        0x80000005 0x003C92 2
                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.12.2    2.2.2.2         1422        0x80000004 0x008922
192.168.23.3    3.3.3.3         1468        0x80000004 0x003C57
192.168.34.4    4.4.4.4         1559        0x80000004 0x00EE8C
            OSPF Router with ID (11.11.11.11) (Process ID 2)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
5.5.5.5         5.5.5.5         1495        0x80000008 0x00ACD9 3
6.6.6.6         6.6.6.6         812         0x80000006 0x00285E 2
11.11.11.11     11.11.11.11     593         0x80000006 0x007628 2
                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.15.5    5.5.5.5         1495        0x80000003 0x004E18
192.168.56.5    5.5.5.5         732         0x80000004 0x008CC3
                Summary Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
7.7.7.7         11.11.11.11     1362        0x80000002 0x0085EB
8.8.8.8         11.11.11.11     1362        0x80000002 0x00610B
44.44.44.44     11.11.11.11     1362        0x80000002 0x00CE0F
192.168.47.0    11.11.11.11     1362        0x80000002 0x0006EF
192.168.78.0    11.11.11.11     1362        0x80000002 0x00B91C
//可以看到，当关于VPN A站点2的前缀在PE2（R4）上的时候，它们都还是类型1或者类型2的LSA；
//而关于这些前缀的LSA在PE1（R1）的时候，它们都已经变为了类型3的LSA；
//这说明，PE2（R4）将类型为1和2的LSA转换为类型3的LSA；
//PE1（R1）收到了远程PE2（R4）发来的类型3的LSA，将其装换为OSPF区域间路由（O IA）；
```

查看R6的IP路由表；
```c
R6#show ip route
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
O        5.5.5.5 [110/2] via 192.168.56.5, 01:25:07, FastEthernet0/0
        6.0.0.0/32 is subnetted, 1 subnets
C        6.6.6.6 is directly connected, Loopback0
        7.0.0.0/32 is subnetted, 1 subnets
O IA     7.7.7.7 [110/4] via 192.168.56.5, 00:27:55, FastEthernet0/0
        8.0.0.0/32 is subnetted, 1 subnets
O IA     8.8.8.8 [110/5] via 192.168.56.5, 00:27:55, FastEthernet0/0
        11.0.0.0/32 is subnetted, 1 subnets
O        11.11.11.11 [110/3] via 192.168.56.5, 00:45:36, FastEthernet0/0
        44.0.0.0/32 is subnetted, 1 subnets
O IA     44.44.44.44 [110/3] via 192.168.56.5, 00:27:55, FastEthernet0/0
O     192.168.15.0/24 [110/2] via 192.168.56.5, 01:25:07, FastEthernet0/0
O IA  192.168.47.0/24 [110/3] via 192.168.56.5, 00:27:55, FastEthernet0/0
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.6/32 is directly connected, FastEthernet0/0
O IA  192.168.78.0/24 [110/4] via 192.168.56.5, 00:27:55, FastEthernet0/0
//注意：在OSPF看来，MPLS VPN超骨干网络超越了OSPF骨干区域Area0，MPLS VPN超骨干网络神似OSPF骨干区域Area0，
//但它又和OSPF骨干区域Area0不同，所以PE路由器会执行ABR的功能，会将类型为1和2的LSA转换为类型3的LSA；
//所以当PE在收到来自对端远程PE发来的OSPF区域内路由（O）时，会将OSPF区域内路由转换为OSPF区域间路由（O IA）；
```

调试OSPF 2的LSA生成；
```c
R1#debug ip ospf 2 lsa-generation
OSPF LSA generation debugging is on for process 2
R1#clear ip ospf 2 process
Reset OSPF process 2? [no]: yes
*Mar 18 18:00:26.607: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.611: %OSPF-5-ADJCHG: Process 2, Nbr 5.5.5.5 on FastEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
*Mar 18 18:00:26.619: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.619: OSPF-2 LSGEN: Scheduling network LSA on FastEthernet0/1
*Mar 18 18:00:26.623: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.627: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.631: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.647: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.651: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.655: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.731: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:26.743: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:27.107: OSPF-2 LSGEN: Build router LSA for area 0, router ID 11.11.11.11, seq 0x80000001
*Mar 18 18:00:27.143: %OSPF-5-ADJCHG: Process 2, Nbr 5.5.5.5 on FastEthernet0/1 from LOADING to FULL, Loading Done
*Mar 18 18:00:27.147: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
*Mar 18 18:00:27.647: OSPF-2 LSGEN: Rate limit LSA generation for 11.11.11.11 11.11.11.11 1
*Mar 18 18:00:27.747: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 7.7.7.7, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:27.751: OSPF-2 LSGEN: Update summary LSA 7.7.7.7 11.11.11.11 3 80000001
*Mar 18 18:00:27.755: OSPF-2 LSGEN: Rate limit LSA generation for 7.7.7.7 11.11.11.11 3
*Mar 18 18:00:27.759: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 8.8.8.8, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:27.763: OSPF-2 LSGEN: Update summary LSA 8.8.8.8 11.11.11.11 3 80000001
*Mar 18 18:00:27.767: OSPF-2 LSGEN: Rate limit LSA generation for 8.8.8.8 11.11.11.11 3
*Mar 18 18:00:27.771: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 44.4
R1#4.44.44, Adv rtr 11.11.11.11, age 3600, seq 0x80000005, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:27.775: OSPF-2 LSGEN: Update summary LSA 44.44.44.44 11.11.11.11 3 80000005
*Mar 18 18:00:27.779: OSPF-2 LSGEN: Rate limit LSA generation for 44.44.44.44 11.11.11.11 3
*Mar 18 18:00:27.783: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 192.168.47.0, Adv rtr 11.11.11.11, age 3600, seq 0x80000005, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:27.787: OSPF-2 LSGEN: Update summary LSA 192.168.47.0 11.11.11.11 3 80000005
*Mar 18 18:00:27.791: OSPF-2 LSGEN: Rate limit LSA generation for 192.168.47.0 11.11.11.11 3
*Mar 18 18:00:27.795: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 192.168.78.0, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:27.799: OSPF-2 LSGEN: Update summary LSA 192.168.78.0 11.11.11.11 3 80000001
*Mar 18 18:00:27.803: OSPF-2 LSGEN: Rate limit LSA generation for 192.168.78.0 11.11.11.11 3
*Mar 18 18:00:28.611: OSPF-2 LSGEN: Scheduling rtr LSA for area 0
R1#
*Mar 18 18:00:29.111: OSPF-2 LSGEN: Rate limit LSA generation for 11.11.11.11 11.11.11.11 1
R1#
*Mar 18 18:00:31.779: OSPF-2 LSMAX: Rcv Maxage LSA, Type 1, LSID 11.11.11.11, Adv rtr 11.11.11.11, age 3600, seq 0x80000006, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:31.787: OSPF-2 LSGEN: Update router LSA 11.11.11.11 11.11.11.11 1 80000006
*Mar 18 18:00:31.791: OSPF-2 LSGEN: Rate limit LSA generation for 11.11.11.11 11.11.11.11 1
*Mar 18 18:00:32.071: OSPF-2 LSMAX: Rcv Maxage LSA, Type 1, LSID 11.11.11.11, Adv rtr 11.11.11.11, age 3600, seq 0x80000006, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:32.075: OSPF-2 LSGEN: Update router LSA 11.11.11.11 11.11.11.11 1 80000006
*Mar 18 18:00:32.079: OSPF-2 LSGEN: Rate limit LSA generation for 11.11.11.11 11.11.11.11 1
R1#
*Mar 18 18:00:32.107: OSPF-2 LSGEN: Build router LSA for area 0, router ID 11.11.11.11, seq 0x80000007
*Mar 18 18:00:32.715: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 7.7.7.7, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:32.719: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 8.8.8.8, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:32.723: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 44.44.44.44, Adv rtr 11.11.11.11, age 3600, seq 0x80000005, from 5.5.5.5, FastEthernet0/1
R1#
*Mar 18 18:00:32.727: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 192.168.47.0, Adv rtr 11.11.11.11, age 3600, seq 0x80000005, from 5.5.5.5, FastEthernet0/1
*Mar 18 18:00:32.727: OSPF-2 LSMAX: Rcv Maxage LSA, Type 3, LSID 192.168.78.0, Adv rtr 11.11.11.11, age 3600, seq 0x80000001, from 5.5.5.5, FastEthernet0/1
//可以看到，正如之前的结论，PE1（R1）收到关于VPN A站点2中前缀的LSA是类型3的LSA；
//证实了PE2（R4）将这些类型为1和2的LSA转换为类型3的LSA后，再将其发送给PE1（R1）；
```

## 实验2：OSPF Sham Link in MPLS VPN
### 理论概述：
在OSPF看来，MPLS VPN超骨干网络超越了OSPF骨干区域Area0，MPLS VPN超骨干网络神似OSPF骨干区域Area0，
但它又和OSPF骨干区域Area0不同，所以PE路由器会执行ABR的功能，会将类型为1和2的LSA转换为类型3的LSA；
所以当PE在收到来自对端远程PE发来的OSPF区域内路由（O）时，会将OSPF区域内路由转换为OSPF区域间路由（O IA）。
为了避免PE将类型1和类型2的LSA转换为类型3的LSA，可以在PE之间创建OSPF Sham Link；这样，当LSA在Sham Link中
进行洪泛时，所有的OSPF路由类型都不会改变，不会转变为类型3或者类型5的LSA；
        
### 配置：
在PE上配置OSPF Sham Link；
```c
R1(config)#router ospf 2 vrf A-Site1
R1(config-router)#area 0 sham-link 11.11.11.11 44.44.44.44
//注意：必须将Sham Link的源地址和目的地址所在的接口划分进VRF；

R4(config)#router ospf 2 vrf A-Site2
R4(config-router)#area 0 sham-link 44.44.44.44 11.11.11.11
```

**注意：在OSPF VRF进程中配置完sham link后发现sham link反复震荡；**
```c
*Mar 20 15:39:37.859: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:39:42.979: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
R4(config)# 
*Mar 20 15:39:48.703: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:39:52.999: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
R4(config)#
*Mar 20 15:39:58.743: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:40:03.011: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
R4(config)#
*Mar 20 15:40:08.879: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:40:13.039: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
//注意：必须在PE上将各自用作Sham Link的源地址的接口地址通告进MP-BGP的IPv4 VRF地址族，否则Sham Link会反复震荡；
```

将接口Loopback1的地址以路由的形式通告进MP-BGP的IPv4 VRF地址族；
```c
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
R1(config-router-af)#network 11.11.11.11 mask 255.255.255.255

R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#network 44.44.44.44 mask 255.255.255.255
```

注意：在将接口Loopback1的地址以路由的形式通告进MP-BGP的IPv4 VRF地址族，发现Sham Link依旧反复震荡；
```c
R4(config)#
*Mar 20 15:39:58.743: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:40:03.011: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
R4(config)#
*Mar 20 15:40:08.879: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
R4(config)#
*Mar 20 15:40:13.039: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from FULL to DOWN, Neighbor Down: Interface down or detached
//注意：作为Sham Link源地址和目的地址的接口不能被通告进OSPF VRF进程；
```
            
将作为Sham Link源地址，并且同时也作为了对方PE Sham Link目的地址的接口移除OSPF VRF进程；
```c
R1(config)#router ospf 2 vrf A-Site1               
R1(config-router)#no network 11.11.11.11 0.0.0.0 area 0

R4(config)#router ospf 2 vrf A-Site2
R4(config-router)#no network 44.44.44.44 0.0.0.0 area 0
```

**OSPF Sham Link持续稳定！**
```c
R1(config-router)#
*Mar 20 16:19:35.779: %OSPF-5-ADJCHG: Process 2, Nbr 44.44.44.44 on OSPF_SL0 from LOADING to FULL, Loading Done

R4(config-router)#
*Mar 20 16:19:36.067: %OSPF-5-ADJCHG: Process 2, Nbr 11.11.11.11 on OSPF_SL0 from LOADING to FULL, Loading Done
```

### 验证与调试：
查看R6的路由表；
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
        5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/2] via 192.168.56.5, 00:57:04, FastEthernet0/0
        6.0.0.0/32 is subnetted, 1 subnets
C        6.6.6.6 is directly connected, Loopback0
        7.0.0.0/32 is subnetted, 1 subnets
O        7.7.7.7 [110/5] via 192.168.56.5, 00:10:36, FastEthernet0/0
        8.0.0.0/32 is subnetted, 1 subnets
O        8.8.8.8 [110/6] via 192.168.56.5, 00:10:36, FastEthernet0/0
        11.0.0.0/32 is subnetted, 1 subnets
O E2     11.11.11.11 [110/1] via 192.168.56.5, 00:18:37, FastEthernet0/0
        44.0.0.0/32 is subnetted, 1 subnets
O E2     44.44.44.44 [110/1] via 192.168.56.5, 00:10:36, FastEthernet0/0
O     192.168.15.0/24 [110/2] via 192.168.56.5, 00:57:04, FastEthernet0/0
O     192.168.47.0/24 [110/4] via 192.168.56.5, 00:10:36, FastEthernet0/0
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.6/32 is directly connected, FastEthernet0/0
O     192.168.78.0/24 [110/5] via 192.168.56.5, 00:10:36, FastEthernet0/0
```
注意到，现在关于R6上看到关于站点2的路由就变成了OSPF区域内路由（O）；但是，由于关于11.11.11.11/32和44.44.44.44/32的路由，由于它们并不是在OSPF里进行通告的,而是通告进BGP，然后才被重分发进OSPF的，所以都变成了OSPF外部路由（O E2）；