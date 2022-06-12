---
title: MPLS 实验11：MPLS VPN running eBGP on the PE-CE link
date: 2014-03-19 20:39:51
updated: 2014-03-19 20:39:51
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
```c
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
router bgp 65001
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
router bgp 65002
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
## 实验1：同一VPN的不同站点使用不同的AS号；
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
因为PE1导出的RT为1:1，PE2导入的RT正是PE1的导出RT1:1；所以PE1（R1）的RT导出和PE2（R4）的RT导入能够匹配成功，R4能够学到R1导入到MP-BGP中的VPNv4路由；又因为PE2导出的RT为1:1，PE1导入的RT也正好是PE2的导出RT1:1；所以PE2（R4）的RT导出和PE1（R1）的RT导入能够匹配成对，R1能够学到R4导入到MP-BGP中的VPNv4路由

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

配置PE1（R1）的MP-BGP IPv4 VRF地址族（AF）；
```c
                R1(config)#router bgp 1
                R1(config-router)#address-family ipv4 vrf A-Site1
                R1(config-router-af)#neighbor 192.168.15.5 remote-as 65001
                R1(config-router-af)#neighbor 192.168.15.5 activate 
                //激活与IBGP邻居R4的VPNv4关系；
                
                R1(config-router-af)#redistribute connected
                //如果用户在CE路由器上ping远程网络的另一个VPN站点中的CE或C路由器，
                //为了使其在没有指定其源地址情况下（即默认使用CE路由器出站接口IP地址），Echo Reply包能够有路由并正常返回,
                //将PE路由器的直连路由重分布进MP-BGP的IPv4 VRF AF中；
```

配置PE2（R4）的MP-BGP IPv4 VRF地址族（AF）；
```c
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#neighbor 192.168.47.7 remote-as 65002
R4(config-router-af)#neighbor 192.168.47.7 activate
R4(config-router-af)#redistribute connected
R4(config-router-af)#exit-address-family
```


### 验证与调试:
在VPN A站点1的CE路由器R5上ping远程VPN A站点2的CE路由器R7；
```c
R5#ping 7.7.7.7
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 7.7.7.7, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 100/146/208 ms
//成功！
```

查看R1的IP BGP VPNv4路由表；
```c
R1(config)#do sh ip bgp vpnv4 all
BGP table version is 34, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
        Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:1 (default for vrf A-Site1)
*> 5.5.5.5/32       192.168.15.5             0             0 65001 i
*>i7.7.7.7/32       4.4.4.4                  0    100      0 65002 i
*> 192.168.15.0     0.0.0.0                  0         32768 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
Route Distinguisher: 1:2
*>i7.7.7.7/32       4.4.4.4                  0    100      0 65002 i
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
```

查看R1的vrf路由表；
```c
R1(config)#do sh ip route vrf A-Site1
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
B        5.5.5.5 [20/0] via 192.168.15.5, 00:48:25
        7.0.0.0/32 is subnetted, 1 subnets
B        7.7.7.7 [200/0] via 4.4.4.4, 00:41:20
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.1/32 is directly connected, FastEthernet0/1
B     192.168.47.0/24 [200/0] via 4.4.4.4, 00:38:35
```

查看R5的IP路由表；
```c
R5(config)#do sh ip route
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
B        7.7.7.7 [20/0] via 192.168.15.1, 00:21:34
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.5/32 is directly connected, FastEthernet0/1
B     192.168.47.0/24 [20/0] via 192.168.15.1, 00:22:04
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.5/32 is directly connected, FastEthernet0/0
```

查看R5的BGP表；
```c
R5(config)#do sh ip bgp
BGP table version is 18, local router ID is 5.5.5.5
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
        Network          Next Hop            Metric LocPrf Weight  Path
*> 5.5.5.5/32       0.0.0.0                  0         32768                             i
*> 7.7.7.7/32       192.168.15.1                           0              1 65002 i
r> 192.168.15.0     192.168.15.1             0             0           1 ?
*> 192.168.47.0     192.168.15.1                           0            1 ?
//可以看到前缀7.7.7.7/32的AS_PATH属性中看到起源AS65002；
```
                
## 实验2：同一VPN的不同站点使用相同的AS号（使用as-override）；
修改R7上BGP AS号为65001，使之与站点1的R5的BGP AS号相同；
```c
R7(config)#no router bgp 65002
R7(config)#router bgp 65001
R7(config-router)#bgp router-id 7.7.7.7
R7(config-router)#bgp log-neighbor-changes
R7(config-router)#network 7.7.7.7 mask 255.255.255.255
R7(config-router)#neighbor 192.168.47.4 remote-as 1
R7(config-router)#exit
```

修改PE2（R4）的PE-CE路由配置，将MP-BGP IPv4 VRF A-Site2的EBGP邻居配置进行修改；
```c
R4(config)#router bgp 1
R4(config-router)#address-f ipv4 vrf A-Site2
R4(config-router-af)#no neighbor 192.168.47.7 remote-as 65002
R4(config-router-af)#neighbor 192.168.47.7 remote-as 65001
R4(config-router-af)#neighbor 192.168.47.7 activate
```

验证：查看R5的IP路由表；
```
R5(config)#do sh ip route
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
        192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, FastEthernet0/1
L        192.168.15.5/32 is directly connected, FastEthernet0/1
B     192.168.47.0/24 [20/0] via 192.168.15.1, 00:57:01
        192.168.56.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.56.0/24 is directly connected, FastEthernet0/0
L        192.168.56.5/32 is directly connected, FastEthernet0/0
//没有发现关于前缀7.7.7.7/32的路由；
```

验证：查看R5的BGP表；
```c
R5(config)#do sh ip bgp    
BGP table version is 19, local router ID is 5.5.5.5
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
*> 5.5.5.5/32       0.0.0.0                  0         32768 i
r> 192.168.15.0     192.168.15.1             0             0 1 ?
*> 192.168.47.0     192.168.15.1                           0 1 ?
//也没有发现关于前缀7.7.7.7/32的路由；
//因为EBGP防环机制，当R5收到来自PE1的BGP路由后，由于前缀7.7.7.7/32的BGP路由中AS_PATH属性有和自己AS相同的65001，
//所以将此路由丢弃了；
```

在PE路由器上，针对各自直连到VRF接口的CE路由器EBGP邻居配置`as-override`；
```c
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
R1(config-router-af)#neighbor 192.168.15.5 as-override

R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#neighbor 192.168.47.7 as-override
```

查看R5的BGP表；
```c
R5(config)#do sh ip bgp
BGP table version is 20, local router ID is 5.5.5.5
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
*> 5.5.5.5/32       0.0.0.0                  0         32768 i
*> 7.7.7.7/32       192.168.15.1                           0 1 1 i
r> 192.168.15.0     192.168.15.1             0             0 1 ?
*> 192.168.47.0     192.168.15.1                           0 1 ?
```

查看R7的BGP表；
```c
R7(config)#do sh ip bgp
BGP table version is 5, local router ID is 7.7.7.7
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
Network          Next Hop            Metric LocPrf Weight Path
*> 5.5.5.5/32       192.168.47.4                           0             1 1 i
*> 7.7.7.7/32       0.0.0.0                  0         32768                   i
*> 192.168.15.0     192.168.47.4                                      0 1 ?
r> 192.168.47.0     192.168.47.4             0                       0 1 ?
```
            
            
# 总结：
PE路由器简单地比较CE路由器ASN和as-path中的ASN，如果匹配的话，所有在as-path中的该ASN都会被服务提供商的ASN所代替。

这样一来，远程CE就会接受这些路由了，因为它们在这些BGP路由的as-path中看不到自己的ASN了！但这样一来，对于可能存在环路的保护机制，以及非最优路由的as-path检查机制就失效了。因此，在使用as-override功能进行ASN覆盖的时候，明智的做法是为BGP实施SOO特性。
