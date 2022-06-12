---
title: MPLS 实验6：MPLS VPN running RIPv2 on the PE-CE link
date: 2014-03-14 22:47:51
updated: 2014-03-14 22:47:51
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
!
interface FastEthernet0/0
    ip address 192.168.56.5 255.255.255.0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.15.5 255.255.255.0
    no shutdown
!
router rip
    version 2
    network 5.0.0.0
    network 192.168.15.0
    network 192.168.56.0
    no auto-summary
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
!
interface FastEthernet0/0
    ip address 192.168.56.6 255.255.255.0
    no shutdown
!
router rip
    version 2
    network 6.0.0.0
    network 192.168.56.0
    no auto-summary
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
!
interface FastEthernet0/0
    ip address 192.168.78.7 255.255.255.0
    no shutdown
!
interface FastEthernet0/1
    ip address 192.168.47.7 255.255.255.0
    no shutdown
!
router rip
    version 2
    network 7.0.0.0
    network 192.168.47.0
    network 192.168.78.0
    no auto-summary
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
!
interface FastEthernet0/0
    ip address 192.168.78.8 255.255.255.0
    no shutdown
!
router rip
    version 2
    network 8.0.0.0
    network 192.168.78.0
    no auto-summary
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
1. 配置MP-BGP：
   - 配置建立PE之间的IBGP邻居关系；
   - 启用VPNv4地址族（AF），并激活与其他PE设备的邻居关系；
1. 配置PE-CE路由；
   - 配置IGP，并启用IGP的地址族（AF）；
   - 配置启用MP-BGP IPv4 VRF地址族，然后激活与其他PE路由器的MP-BGP IPv4 VRF邻居关系；

# 实验与调试：
## 实验1：配置MPLS VPN
### **在PE上创建VRF**

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
            
在PE2（R4）上创建VRF，并命名为A-Site2，表示该VRF为VPN A的站点2服务：
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
因为PE1导出的RT为1:1，PE2导入的RT正是PE1的导出RT1:1， 所以PE1（R1）的RT导出和PE2（R4）的RT导入能够匹配成功，R4能够学到R1导入到MP-BGP中的VPNv4路由；又因为PE2导出的RT为1:1，PE1导入的RT也正好是PE2的导出RT1:1；所以PE2（R4）的RT导出和PE1（R1）的RT导入能够匹配成对，R1能够学到R4导入到MP-BGP中的VPNv4路由。

将PE2（R4）上与CE相连的PE接口F0/1关联到VRF A-Site2；
```c
R4(config)#int f0/1
R4(config-if)#ip vrf forwarding A-Site2
% Interface FastEthernet0/1 IPv4 disabled and address(es) removed due to enabling VRF A-Site2
//切记先将接口关联VRF，然后再配置IP地址；
R4(config-if)#ip add 192.168.47.4 255.255.255.0
```
            

### **配置MP-BGP；**
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
                
                
### **配置PE-CE路由；**

配置PE1（R1）的IGP；
```c
R1(config)#router rip   
R1(config-router)#version 2
R1(config-router)#no auto-summary 
```
配置PE1（R1）的IGP地址族（AF）；
```c
//进入IGP RIP的AF模式；
R1(config-router)#address-family ipv4 ?
    unicast  Address Family Modifier
    vrf      Specify parameters for a VPN Routing/Forwarding instance
    <cr>
R1(config-router)#address-family ipv4 vrf A-Site1
R4(config-router-af)#version 2
R4(config-router-af)#no auto-summary
//将IGP RIP的IPv4 VRF地址族（AF）配置为RIPv2,并关闭自动汇总；

R1(config-router-af)#redistribute bgp 1 metric ?
    <0-16>       Default metric
    transparent  Transparently redistribute metric
R1(config-router-af)#redistribute bgp 1 metric transparent 
// 关键字transparent表示透传，RIP将不会察觉到MPLS VPN的存在；
//必须指定Metric否则没有BGP能被重分布进RIP;
//将MP-BGP中IPv4 VRF名字与IGP中IPv4 VRF名字相同的地址族（AF）重分布进IGP的IPv4 VRF AF；
//IGP和MP-BGP中IPv4 VRF地址族（AF）实际上是执行相互重分发的操作；

R1(config-router-af)#network 192.168.15.0 
//将PE连接到CE的VRF接口宣告进IGP进程的IPv4 VRF地址族中；

R1(config-router-af)#exit-address-family
//退出IGP RIP的AF模式；
```
            
配置PE1（R1）的MP-BGP IPv4 VRF地址族（AF）；
```c
R1(config)#router bgp 1
//进入MP-BGP的IPv4 VRF地址族（AF）；
R1(config-router)#address-family ipv4 vrf A-Site1

R1(config-router-af)#redistribute rip 
//将IGP中IPv4 VRF名字与MP-BGP中IPv4 VRF名字相同的地址族（AF）重分布进MP-BGP的IPv4 VRF AF；
//IGP和MP-BGP中IPv4 VRF地址族（AF）实际上是执行相互重分发的操作；

R1(config-router-af)#redistribute connected 
//如果用户在CE路由器上ping远程网络的另一个VPN站点中的CE或C路由器，
//为了使其在没有指定其源地址情况下（即默认使用CE路由器出站接口IP地址），Echo Reply包能够有路由并正常返回,
//将PE路由器的直连路由重分布进MP-BGP的IPv4 VRF AF中；

R1(config-router-af)#exit-address-family 
//退出MP-BGP的IPv4 VRF地址族（AF）模式；
```
                
                
配置PE2（R4）的IGP；
```c
R4(config)#router rip
R4(config-router)#version 2
R4(config-router)#no auto-summary
```

配置PE2（R4）的IGP地址族（AF）；
```c
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#version 2
R4(config-router-af)#no auto-summary
R4(config-router-af)#redistribute bgp 1 metric transparent
R4(config-router-af)#network 192.168.47.0
R4(config-router-af)#exit-address-family
```
            
配置PE2（R4）的MP-BGP IPv4 VRF地址族（AF）；
```c
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#redistribute connected
R4(config-router-af)#redistribute rip
R4(config-router-af)#exit-address-family
```
                

# 验证与调试：
以ping 8.8.8.8的ICMP包转发交换过程为例，验证并解释MPLS VPN中数据包交换过程；
```c
//在VPN A站点1的C路由器R6上ping远程VPN A站点2的C路由器R8；
R6#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 132/160/196 ms
//成功！
```


VPN A站点1的C路由器R6在发送ICMP echo Request数据包之前查看IP路由表，准确的说不是查看路由表，而是CEF表；
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
R        5.5.5.5 [120/1] via 192.168.56.5, 00:00:08, FastEthernet0/0
        6.0.0.0/32 is subnetted, 1 subnets
C        6.6.6.6 is directly connected, Loopback0
        7.0.0.0/32 is subnetted, 1 subnets
R        7.7.7.7 [120/3] via 192.168.56.5, 00:00:08, FastEthernet0/0
        8.0.0.0/32 is subnetted, 1 subnets
R        8.8.8.8 [120/4] via 192.168.56.5, 00:00:08, FastEthernet0/0
R     192.168.15.0/24 [120/1] via 192.168.56.5, 00:00:08, FastEthernet0/0
R     192.168.47.0/24 [120/2] via 192.168.56.5, 00:00:08, FastEthernet0/0
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.6/32 is directly connected, FastEthernet0/0
R     192.168.78.0/24 [120/3] via 192.168.56.5, 00:00:08, FastEthernet0/0
//R6有关于8.8.8.8/32的路由条目，并且是通过RIP从R5学到的；
//R6将数据包转发给R5；
```

此时Echo Request包达到R5，查看R5的IP路由表；
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
        6.0.0.0/32 is subnetted, 1 subnets
R        6.6.6.6 [120/1] via 192.168.56.6, 00:00:05, FastEthernet0/0
        7.0.0.0/32 is subnetted, 1 subnets
R        7.7.7.7 [120/2] via 192.168.15.1, 00:00:23, FastEthernet0/1
        8.0.0.0/32 is subnetted, 1 subnets
R        8.8.8.8 [120/3] via 192.168.15.1, 00:00:23, FastEthernet0/1
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.5/32 is directly connected, FastEthernet0/1
R     192.168.47.0/24 [120/1] via 192.168.15.1, 00:00:23, FastEthernet0/1
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.5/32 is directly connected, FastEthernet0/0
R     192.168.78.0/24 [120/2] via 192.168.15.1, 00:00:23, FastEthernet0/1
//R5发现自己拥有关于8.8.8.8/32的路由条目，并且此路由条目是通过RIP从PE1（R1）学到的；
//R5将数据包转发给PE1（R1）；
```

**此时Echo Request包抵达PE1（R1）；**

由于从R5转发来的ping包是在接口F0/1上收到的，而此接口又与VRF A-Site1相关联，所以PE1需要查看IP vrf A-Site1路由表；准确的说也不是查看IP vrf A-Site1路由表，而是查看IP CEF vrf A-Site1表，但毕竟CEF表来自于路由表，这里只是为了说明选路的依据；
```c
R1#show ip route vrf A-Site1

Routing Table: A-Site1
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
R        5.5.5.5 [120/1] via 192.168.15.5, 00:00:04, FastEthernet0/1
        6.0.0.0/32 is subnetted, 1 subnets
R        6.6.6.6 [120/2] via 192.168.15.5, 00:00:04, FastEthernet0/1
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [200/1] via 4.4.4.4, 01:40:04
        8.0.0.0/32 is subnetted, 1 subnets
B        8.8.8.8 [200/2] via 4.4.4.4, 01:40:04
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.1/32 is directly connected, FastEthernet0/1
B     192.168.47.0/24 [200/0] via 4.4.4.4, 01:40:04
R     192.168.56.0/24 [120/1] via 192.168.15.5, 00:00:04, FastEthernet0/1
B     192.168.78.0/24 [200/1] via 4.4.4.4, 01:40:04
```

PE1（R1）发现自己拥有关于8.8.8.8/32的路由条目，并且此路由条目是通过IBGP从R4（PE2）学到的；与纯MPLS网络中一样，去往8.8.8.8/32的路由需要递归查找明确的下一跳；要达到8.8.8.8/32必须先到达4.4.4.4/32，因为4.4.4.4/32不是VPNv4路由，所以4.4.4.4/32的路由存在于普通IP路由表中；

因而要达到4.4.4.4/32就必须查看普通IP路由表：
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
O        2.2.2.2 [110/2] via 192.168.12.2, 02:08:31, FastEthernet0/0
        3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/3] via 192.168.12.2, 02:08:21, FastEthernet0/0
        4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/4] via 192.168.12.2, 02:08:21, FastEthernet0/0
        192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, FastEthernet0/0
L        192.168.12.1/32 is directly connected, FastEthernet0/0
O     192.168.23.0/24 [110/2] via 192.168.12.2, 02:08:21, FastEthernet0/0
O     192.168.34.0/24 [110/3] via 192.168.12.2, 02:08:21, FastEthernet0/0
```
要去往4.4.4.4/32就必须到达192.168.12.2，至此PE1发现了关于8.8.8.8/32明确的下一跳；

由于关于8.8.8.8/32的BGP路由器条目是一条VPNv4路由，所以查看PE1的IP BGP VPNv4表；
```c
R1#show ip bgp vpnv4 all
BGP table version is 13, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

    Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:1 (default for vrf A-Site1)
*> 5.5.5.5/32       192.168.15.5             1         32768 ?
*> 6.6.6.6/32       192.168.15.5             2         32768 ?
*>i7.7.7.7/32       4.4.4.4                  1    100      0 ?
*>i8.8.8.8/32       4.4.4.4                  2    100      0 ?
*> 192.168.15.0     0.0.0.0                  0         32768 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*> 192.168.56.0     192.168.15.5             1         32768 ?
*>i192.168.78.0     4.4.4.4                  1    100      0 ?
Route Distinguisher: 1:2
*>i7.7.7.7/32       4.4.4.4                  1    100      0 ?
*>i8.8.8.8/32       4.4.4.4                  2    100      0 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*>i192.168.78.0     4.4.4.4                  1    100      0 ?
```

查看PE1的IP BGP VPNv4表明细路由；
```c
R1#show ip bgp vpnv4 rd 1:1 8.8.8.8
BGP routing table entry for 1:1:8.8.8.8/32, version 11
Paths: (1 available, best #1, table A-Site1)
    Not advertised to any peer
    Local, imported path from 1:2:8.8.8.8/32
    4.4.4.4 (metric 4) from 4.4.4.4 (4.4.4.4)
        Origin incomplete, metric 2, localpref 100, valid, internal, best
        Extended Community: RT:1:1
        mpls labels in/out nolabel/22//标签22是远程PE2（R4）MP-BGP通告的VPNv4路由中包含的VPNv4标签，此标签被用作MPLS VPN的栈底标签；

R1#show ip bgp vpnv4 rd 1:2 8.8.8.8
BGP routing table entry for 1:2:8.8.8.8/32, version 7
Paths: (1 available, best #1, no table)
    Not advertised to any peer
    Local
    4.4.4.4 (metric 4) from 4.4.4.4 (4.4.4.4)
        Origin incomplete, metric 2, localpref 100, valid, internal, best
        Extended Community: RT:1:1
        mpls labels in/out nolabel/22//标签22是远程PE2（R4）MP-BGP通告的VPNv4路由中包含的VPNv4标签，此标签被用作MPLS VPN的栈底标签；
```

查看LFIB表中关于8.8.8.8/32的详细信息；
```c
R1#show mpls forwarding-table vrf A-Site1 8.8.8.8 detail
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
None       22         8.8.8.8/32[V]    0             Fa0/0      192.168.12.2
        MAC/Encaps=14/22, MRU=1496, Label Stack{17 22} //标签栈中有两个标签，标签17用于在AS1中转发数据，栈底标签22为VPNv4标签；
        CA0518300008CA04183000088847 0001100000016000
        VPN route: A-Site1
        No output feature configured
```

知道了前缀8.8.8.8/32的递归下一跳之后，沿递归路径查看每一跳的标签信息；查看PE1的IP CEF VRF A-Site1表中8.8.8.8的详细信息；
```c
R1#sh ip cef vrf A-Site1 8.8.8.8 detail
8.8.8.8/32, epoch 0, flags rib defined all labels
    recursive via 4.4.4.4 label 22//标签22为栈底的VPNv4标签；
    nexthop 192.168.12.2 FastEthernet0/0 label 17//8.8.8.8/32递归"继承了"4.4.4.4的标签17；
```

查看PE1（R1）的IP CEF表中4.4.4.4的详细信息；
```c
R1#sh ip cef 4.4.4.4 detail
4.4.4.4/32, epoch 0
    local label info: global/18
    1 RR source [no flags]
    nexthop 192.168.12.2 FastEthernet0/0 label 17
//标签17为PE1（R1）关于前缀4.4.4.4/32的出站标签，也就是R2通告给R1的入站标签；
```

查看PE1（R1）的IP CEF表中192.168.12.2的详细信息；
```c
R1#show ip cef 192.168.12.2 detail
192.168.12.2/32, epoch 0, flags attached
    Adj source: IP adj out of FastEthernet0/0, addr 192.168.12.2 6819D980
    Dependent covered prefix type adjfib cover 192.168.12.0/24
    attached to FastEthernet0/0
//由于R1与192.168.12.0/24直连，所以不再需要在进行标签转发，所以R1没有关于192.168.12.0/24的出标签；
```

至此，PE1（R1）为数据包打上出标签17，并将数据转发给R2；
```c
//此时ping包抵达R2；
//查看R2的LFIB表；
R2#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  1.1.1.1/32       20757         Fa0/0      192.168.12.1
17         16         4.4.4.4/32       22341         Fa0/1      192.168.23.3
18         Pop Label  3.3.3.3/32       0             Fa0/1      192.168.23.3
19         Pop Label  192.168.34.0/24  0             Fa0/1      192.168.23.3
```
在收到R1转发来的带有R2本地标签17的数据包后，R2使用为数据包打上出标签16，并将其转发给R3；

**注意：递归查找路由表只会发生在运行BGP并与其他PE路由器建立IBGP关系的PE路由器，而在P路由器上，由于没有运行BGP，不可能有相关路由信息，所以只按照标签转发。**

此时ping包抵达R3；
```c
//查看R3的LFIB表；
R3#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  4.4.4.4/32       27288         Fa0/0      192.168.34.4
17         Pop Label  2.2.2.2/32       0             Fa0/1      192.168.23.2
18         16         1.1.1.1/32       29345         Fa0/1      192.168.23.2
19         Pop Label  192.168.12.0/24  0             Fa0/1      192.168.23.2
//R3弹出栈顶标签，并将数据转发给PE2（R4）；
```

此时ping包抵达PE2（R4），由于数据中栈顶的标签被弹出，所以原本栈底的VPNv4标签就变成了栈顶标签；
```c
//查看PE2（R4）的LFIB表；
R4#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  3.3.3.3/32       0             Fa0/0      192.168.34.3
17         Pop Label  192.168.23.0/24  0             Fa0/0      192.168.34.3
18         17         2.2.2.2/32       0             Fa0/0      192.168.34.3
19         18         1.1.1.1/32       0             Fa0/0      192.168.34.3
20         19         192.168.12.0/24  0             Fa0/0      192.168.34.3
21         No Label   7.7.7.7/32[V]    0             Fa0/1      192.168.47.7
22         No Label   8.8.8.8/32[V]    570           Fa0/1      192.168.47.7
23         No Label   192.168.47.0/24[V]   \
                                        0             aggregate/A-Site2 
24         No Label   192.168.78.0/24[V]   \
                                        0             Fa0/1      192.168.47.7
```
R4收到携带VPNv4标签22的Echo Request数据后，移除Echo Request数据的标签栈，将其转发给R7。

数据到达R7后，按照正常的转发方式被转发到R8。
```c
R7#show ip route 8.8.8.8
Routing entry for 8.8.8.8/32
    Known via "rip", distance 120, metric 1
    Redistributing via rip
    Last update from 192.168.78.8 on FastEthernet0/0, 00:00:07 ago
    Routing Descriptor Blocks:
    * 192.168.78.8, from 192.168.78.8, 00:00:07 ago, via FastEthernet0/0
        Route metric is 1, traffic share count is 1
R7#show ip cef 8.8.8.8 detail 
8.8.8.8/32, epoch 0
    nexthop 192.168.78.8 FastEthernet0/0
```

最终，ICMP Echo Request达到了R8，返回R6的ICMP Echo Reply的方向相反，但是过程和操作是一样的。