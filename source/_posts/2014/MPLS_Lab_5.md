---
title: MPLS 实验5：MPLS VPN using Static Routing on the PE-CE link
date: 2014-03-14 22:46:51
updated: 2014-03-14 22:46:51
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
ip route 6.6.6.6 255.255.255.255 192.168.56.6
ip route 0.0.0.0 0.0.0.0 192.168.15.1
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
ip route 0.0.0.0 0.0.0.0 192.168.56.5
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
ip route 0.0.0.0 0.0.0.0 192.168.47.4
ip route 8.8.8.8 255.255.255.255 192.168.78.8
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
ip route 0.0.0.0 0.0.0.0 192.168.78.7
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
//这里因为PE1（R1）的RT导出和PE2（R4）的RT导入能够匹配成功，所以R4能够学到R1导入到MP-BGP中的VPNv4路由；
//又因为PE2（R4）的RT导出和PE1（1）的RT导入能够匹配成对，所以R1能够学到R4导入到MP-BGP中的VPNv4路由；
```

将PE2（R4）上与CE相连的PE接口F0/1关联到VRF A-Site2；
```c
R4(config)#int f0/1
R4(config-if)#ip vrf forwarding A-Site2
% Interface FastEthernet0/1 IPv4 disabled and address(es) removed due to enabling VRF A-Site2
    //切记先将接口关联VRF，然后再配置IP地址；
R4(config-if)#ip add 192.168.47.4 255.255.255.0
```
            

### **配置MP-BGP**

在PE1（R1）上配置MP-BGP，使之与其他PE建立IBGP关系，在此实验中只有PE1和PE2两台PE设备；
```c
R1(config)#router bgp 1
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
R4(config-router)#nei 1.1.1.1 remote-as 1
R4(config-router)#neighbor 1.1.1.1 update-source l0
R4(config-router)#neighbor 1.1.1.1 next-hop-self
```

启用PE2（R4）的VPNv4地址族，并激活与IBGP邻居PE1（R1）的MP-BGP VPNv4邻居关系；
```c
R4(config-router)#address-family vpnv4
R4(config-router-af)#neighbor 1.1.1.1 activate
R4(config-router-af)#neighbor 1.1.1.1 send-community both
```
            
 
### **配置PE-CE路由**
配置PE1（R1）的静态路由；
```c
R1(config)#ip route vrf A-Site1 6.6.6.6 255.255.255.255 192.168.15.5
```

配置PE1（R1）的MP-BGP IPv4 VRF地址族（AF）；
```c
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
//进入MP-BGP的IPv4 VRF地址族（AF）；

R1(config-router-af)#redistribute static 
//将静态路由中VRF名字与MP-BGP中IPv4 VRF名字相同的静态路由重分布进MP-BGP的IPv4 VRF AF；

R1(config-router-af)#redistribute connected 
//如果用户在CE路由器上ping远程网络的另一个VPN站点中的CE或C路由器，
//为了使其在没有指定其源地址情况下（即默认使用CE路由器出站接口IP地址），Echo Reply包能够有路由并正常返回,
//将PE路由器的直连路由重分布进MP-BGP的IPv4 VRF AF中；

R1(config-router-af)#exit-address-family 
//退出MP-BGP的IPv4 VRF地址族（AF）模式；
```
                
                
配置PE2（R4）的静态路由；
```c
R1(config)#ip route vrf A-Site1 6.6.6.6 255.255.255.255 192.168.15.5
```

配置PE2（R4）的MP-BGP IPv4 VRF地址族（AF）；
```c
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#redistribute connected
R4(config-router-af)#redistribute static
R4(config-router-af)#exit-address-family
```

# 实验与调试
在VPN A站点1的C路由器R6上ping远程VPN A站点2的C路由器R8，并且指定源为R6的接口Loopback0；
```c
R6(config)#do p 8.8.8.8 source 6.6.6.6
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
Packet sent with a source address of 6.6.6.6 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 128/159/200 ms
```
成功！