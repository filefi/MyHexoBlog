---
title: MPLS 实验7：MPLS VPN running EIGRP on the PE-CE link
date: 2014-03-14 22:48:51
updated: 2014-03-14 22:48:51
tags: [MPLS, Network]
categories: Network
---


<!-- more -->

# 实验环境：
- 模拟器：GNS3 0.8.6
- Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image
    
# GNS3实验拓扑文件：
[拓扑文件](topology.net)


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
router eigrp 1
    network 5.5.5.5 0.0.0.0
    network 192.168.15.5 0.0.0.0
    network 192.168.56.0
    eigrp router-id 5.5.5.5
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
router eigrp 1
    network 6.6.6.6 0.0.0.0
    network 192.168.56.0
    eigrp router-id 6.6.6.6
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
router eigrp 1
    network 7.7.7.7 0.0.0.0
    network 192.168.47.0
    network 192.168.78.0
    eigrp router-id 7.7.7.7
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
router eigrp 1
    network 8.8.8.8 0.0.0.0
    network 192.168.78.0
    eigrp router-id 8.8.8.8
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
2. 配置MP-BGP：
   - 配置建立PE之间的IBGP邻居关系；
   - 启用VPNv4地址族（AF），并激活与其他PE设备的邻居关系；
3. 配置PE-CE路由；
   - 配置IGP，并启用IGP的地址族（AF）；
   - 配置启用MP-BGP IPv4 VRF地址族，然后激活与其他PE路由器的MP-BGP IPv4 VRF邻居关系；


# 实验与调试：
 
## 实验1：不同的VPN站点之间使用的EIGRP AS号相同；

### 实验1拓扑:
![](topo1.png)

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
R1(config-vrf)#route-target export 1:1
R1(config-vrf)#route-target import 1:2
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

//指定RD，为1:2，如果按照AS：num命名法，以下RD表示AS1中的第2个VRF；
R4(config-vrf)#rd 1:2

//指定RT的导入和导出，这里我将RT指定为导入和导出1：1；
R4(config-vrf)#route-target export 1:2
R4(config-vrf)#route-target import 1:1
```
因为PE1导出的RT为1:1，PE2导入的RT正是PE1的导出RT1:1；所以PE1（R1）的RT导出和PE2（R4）的RT导入能够匹配成功，R4能够学到R1导入到MP-BGP中的VPNv4路由；又因为PE2导出的RT为1:2，PE1导入的RT正是PE2的导出RT1:2；所以PE2（R4）的RT导出和PE1（R1）的RT导入能够匹配成对，R1能够学到R4导入到MP-BGP中的VPNv4路由；

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
                
                
### **配置PE-CE路由**
配置PE1（R1）的IGP；
```c
R1(config)#router eigrp 1
R1(config-router)#no auto-summary
```

配置PE1（R1）的IGP地址族（AF）；
```c
R1(config-router)#address-family ipv4 vrf A-Site1 ?                   
    autonomous-system  Specify Address-Family Autonomous System Number
    <cr>

R1(config-router)#address-family ipv4 vrf A-Site1 autonomous-system 1
//进入IGP EIGRP的AF模式，切记为EIGRP ipv4 vrf地址族（AF）指定一个EIGRP自治系统号；
//由于在实施MPLS VPN，ISP和VPN客服站点本地可能就已经部署了EIGRP，并为EIGRP指定了不同的AS号,
//而在实施MPLS VPN将PE和站点的CE连接起来的时候，由于PE和CE部署EIGRP时没有使用相同的AS号，
//所以PE和CE无法建立EIGRP邻居关系；
//为避免修改原始EIGRP的AS号，因此，在EIGRP的ipv4 vrf地址族（AF）里为PE和CE指定相同的EIGRP AS号；

R4(config-router-af)#no auto-summary
//将IGP EIGRP的IPv4 VRF地址族（AF）关闭自动汇总；

R1(config-router-af)#redistribute bgp 1 metric 10000 100 255 1 1500
//必须指定Metric;
//将MP-BGP中IPv4 VRF名字与IGP中IPv4 VRF名字相同的地址族（AF）重分布进IGP的IPv4 VRF AF；
//IGP和MP-BGP中IPv4 VRF地址族（AF）实际上是执行相互重分发的操作；

R1(config-router-af)#network 192.168.15.0 0.0.0.255 
//将PE连接到CE的VRF接口宣告进IGP EIGRP进程的IPv4 VRF地址族中；

R1(config-router-af)#exit-address-family
//退出IGP EIGRP的AF模式；
```
            
配置PE1（R1）的MP-BGP IPv4 VRF地址族（AF）；
```c
R1(config)#router bgp 1
R1(config-router)#address-family ipv4 vrf A-Site1
//进入MP-BGP的IPv4 VRF地址族（AF）；

R1(config-router-af)#redistribute eigrp 1 
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
R4(config)#router eigrp 1
R4(config-router)#no auto-summary

//配置PE2（R4）的IGP地址族（AF）；
R4(config-router)#address-family ipv4 vrf A-Site2 autonomous-system 1
R4(config-router-af)#no auto-summary
R4(config-router-af)#redistribute bgp 1 metric 10000 100 255 1 1500
R4(config-router-af)#network 192.168.47.0 0.0.0.255
R4(config-router-af)#exit-address-family


//配置PE2（R4）的MP-BGP IPv4 VRF地址族（AF）；
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#redistribute connected
R4(config-router-af)#redistribute eigrp 1
R4(config-router-af)#exit-address-family
```
        
### 验证与调试：
在VPN A站点1的C路由器R6上ping远程VPN A站点2的C路由器R8；
```c
R6(config-router)#do p 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 132/162/232 ms
//成功！
```

查看R1的IP BGP VPNv4路由表；
```c
R1(config-vrf)#do sh ip bgp vpnv4 all
BGP table version is 86, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
                                                                                
    Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:1 (default for vrf A-Site1)
*> 5.5.5.5/32       192.168.15.5        156160         32768 ?
*> 6.6.6.6/32       192.168.15.5        158720         32768 ?
*>i7.7.7.7/32       4.4.4.4             156160    100      0 ?
*>i8.8.8.8/32       4.4.4.4             158720    100      0 ?
*> 192.168.15.0     0.0.0.0                  0         32768 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*> 192.168.56.0     192.168.15.5         30720         32768 ?
*>i192.168.78.0     4.4.4.4         4278221055    100      0 ?
Route Distinguisher: 1:2
*>i7.7.7.7/32       4.4.4.4             156160    100      0 ?
*>i8.8.8.8/32       4.4.4.4             158720    100      0 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*>i192.168.78.0     4.4.4.4         4278221055    100      0 ?
```

删除R1上ip vrf A-Site1的RT导入1：2；
```c
R1(config)#ip vrf A-Site1
R1(config-vrf)#no route-target import 1:2
```

查看R1的IP BGP VPNv4路由表；
```c
R1(config-vrf)#do sh ip bgp vpnv4 all
BGP table version is 78, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
                                                                                
    Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:1 (default for vrf A-Site1)
*> 5.5.5.5/32       192.168.15.5        156160         32768 ?
*> 6.6.6.6/32       192.168.15.5        158720         32768 ?
*> 192.168.15.0     0.0.0.0                  0         32768 ?
*> 192.168.56.0     192.168.15.5         30720         32768 ?
//关于RT1:2的VPNv4路由消失了！
```

让R1再度导入RT1:2的VPNv4路由；
```c
R1(config-vrf)#route-target import 1:2
```

查看R1的IP BGP VPNv4路由表；
```c
R1(config-vrf)#do sh ip bgp vpnv4 all
BGP table version is 86, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete
                                                                                
    Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:1 (default for vrf A-Site1)
*> 5.5.5.5/32       192.168.15.5        156160         32768 ?
*> 6.6.6.6/32       192.168.15.5        158720         32768 ?
*>i7.7.7.7/32       4.4.4.4             156160    100      0 ?
*>i8.8.8.8/32       4.4.4.4             158720    100      0 ?
*> 192.168.15.0     0.0.0.0                  0         32768 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*> 192.168.56.0     192.168.15.5         30720         32768 ?
*>i192.168.78.0     4.4.4.4         4278221055    100      0 ?
Route Distinguisher: 1:2
*>i7.7.7.7/32       4.4.4.4             156160    100      0 ?
*>i8.8.8.8/32       4.4.4.4             158720    100      0 ?
*>i192.168.47.0     4.4.4.4                  0    100      0 ?
*>i192.168.78.0     4.4.4.4         4278221055    100      0 ?
//关于RT1:2的VPNv4路由又回来了！
```
        
将R1的EIGRP ip vrf A-Site1的AS号1改为2；
```c
R1(config-router)#no address-family ipv4 vrf A-Site1 autonomous-system 1
R1(config-router)#address-family ipv4 vrf A-Site1 autonomous-system 2
R1(config-router-af)#  redistribute bgp 1 metric 10000 100 255 1 1500
R1(config-router-af)#  network 192.168.15.0
R1(config-router-af)# exit-address-family
//仅仅修改AS号，其他配置都不变；
```


查看EIGRP vrf A-Site1的邻居表；
```c
R1#sh ip eigrp vrf A-Site1 neighbors 
EIGRP-IPv4 Neighbors for AS(2) VRF(A-Site1)
//由于现在PE1的EIGRP ip vrf A-Site1的AS号为2，而CE1的EIGRP AS号为1，所以PE1和CE1无法建立EIGRP邻居；
```

将配置修改回来；
```c
R1(config)#router eigrp 1
R1(config-router)#no auto-summary
R1(config-router)#no address-family ipv4 vrf A-Site1 auto 2
R1(config-router)#address-family ipv4 vrf A-Site1 auto 1
R1(config-router-af)#no auto-summary
R1(config-router-af)#redistribute bgp 1 metric 10000 100 255 1 1500
R1(config-router-af)#network 192.168.15.0 0.0.0.255
R1(config-router-af)#exit
R1(config-router)#exit
*Mar 18 11:30:00.263: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.15.5 (FastEthernet0/1) is up: new adjacency
R1(config)#router bgp 1
R1(config-router)# address-family ipv4 vrf A-Site1
R1(config-router-af)#redistribute connected
R1(config-router-af)# redistribute eigrp 1
R1(config-router-af)#exit-address-family
R1(config-router)#exit
```
可以看到，PE1和CE1的EIGRP邻居关系又建立了！
                
                
## 实验2：将R8配置为
查看R1的IP BGP VPNv4路由表中关于前缀8.8.8.8/32的详细信息；
```c
R1#sh ip bgp vpnv4 vrf A-Site1 8.8.8.8
BGP routing table entry for 1:1:8.8.8.8/32, version 114
Paths: (1 available, best #1, table A-Site1)
    Not advertised to any peer
    Local, imported path from 1:2:8.8.8.8/32
    4.4.4.4 (metric 4) from 4.4.4.4 (4.4.4.4)
        Origin incomplete, metric 158720, localpref 100, valid, internal, best
        Extended Community: RT:1:2 Cost:pre-bestpath:128:158720 0x8800:32768:0 
        0x8801:1:133120 0x8802:65282:25600 0x8803:65281:1500 
        0x8806:0:134744072
        mpls labels in/out nolabel/21
```

为了在VPN A站点2 C2路由器R8上创造一条EIGRP外部路由，不使用network命令宣告Loopback0口，而通过将直连接口Loopback0重分发进EIGRP中；
```c
R8(config)#route-map LOOPBACK permit 10
R8(config-route-map)#match interface loopback 0

R8(config)#router eigrp 1
R8(config-router)#no network 8.8.8.8 0.0.0.0
R8(config-router)#redistribute connected route-map LOOPBACK
``` 
        
查看CE2（R7）的IP路由表；
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
D        5.5.5.5 [90/158720] via 192.168.47.4, 00:11:01, FastEthernet0/1
        6.0.0.0/32 is subnetted, 1 subnets
D        6.6.6.6 [90/161280] via 192.168.47.4, 00:11:01, FastEthernet0/1
        7.0.0.0/32 is subnetted, 1 subnets
C        7.7.7.7 is directly connected, Loopback0
        8.0.0.0/32 is subnetted, 1 subnets
D EX     8.8.8.8 [170/156160] via 192.168.78.8, 00:01:46, FastEthernet0/0
D     192.168.15.0/24 [90/30720] via 192.168.47.4, 00:11:01, FastEthernet0/1
        192.168.47.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.47.0/24 is directly connected, FastEthernet0/1
L        192.168.47.7/32 is directly connected, FastEthernet0/1
D     192.168.56.0/24 [90/33280] via 192.168.47.4, 00:11:01, FastEthernet0/1
        192.168.78.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.78.0/24 is directly connected, FastEthernet0/0
L        192.168.78.7/32 is directly connected, FastEthernet0/0
//可以看到前缀8.8.8.8/32现在以EIGRP外部路由出现在了R7的路由表中；
```
    
此时再来查看R1的IP BGP VPNv4路由表中关于前缀8.8.8.8/32的详细信息；
```c
R1#sh ip bgp vpnv4 vrf A-Site1 8.8.8.8
BGP routing table entry for 1:1:8.8.8.8/32, version 150
Paths: (1 available, best #1, table A-Site1)
    Not advertised to any peer
    Local, imported path from 1:2:8.8.8.8/32
    4.4.4.4 (metric 4) from 4.4.4.4 (4.4.4.4)
        Origin incomplete, metric 158720, localpref 100, valid, internal, best
        Extended Community: RT:1:2 Cost:pre-bestpath:129:158720 0x8800:0:0 
        0x8801:1:133120 0x8802:65282:25600 0x8803:65281:1500 
        0x8804:0:134744072 0x8805:11:0 0x8806:0:134744072
        mpls labels in/out nolabel/26

```
- 此时R1的IP BGP VPNv4路由表中关于前缀8.8.8.8/32的详细信息中拓展团体属性中多了消息类型0x8804和0x8805；
- 拓展团体属性中类型0x8804和0x8805的字段是VPNv4路由用来描述远程VPN站点EIGRP外部路由的；
- 仅当在远程站点的网络中该路由条目就已经是EIGRP外部路由时，
- 才会在VPNv4的扩展团体属性中携带类型0x8804和0x8805字段，来为此EIGRP外部路由进- 下面实验3中的场景则不会出现类型0x8804和0x8805字段； 
            

## 实验3：
### 实验3拓扑：

![](topo2.png)


修改VPN A站点2的EIGRP配置，将EIGRP的AS号从1改为2；
```c
R7(config)#no router ei 1
R7(config)#router eigrp 2
R7(config-router)#network 7.7.7.7 0.0.0.0
R7(config-router)#network 192.168.47.0
R7(config-router)#network 192.168.78.0
R7(config-router)#eigrp router-id 7.7.7.7
R7(config-router)#no au

R8(config)#no router ei 1
R8(config)#router eigrp 2
R8(config-router)#no redistribute connected route-map LOOPBACK
R8(config-router)#network 8.8.8.8 0.0.0.0
R8(config-router)#network 192.168.78.0
R8(config-router)#eigrp router-id 8.8.8.8
R8(config-router)#no auto
```
            
修改PE2（R4）上EIGRP 1中的ipv4 vrf A-Site1地址族的AS号2；
```c
R4(config)#router eigrp 1
R4(config-router)#no address-family ip vrf A-Site2 autonomous-system 1
R4(config-router)#address-family ip vrf A-Site2 autonomous-system 2
R4(config-router-af)#no auto-summary
R4(config-router-af)#redistribute bgp 1 metric 10000 100 255 1 1500
R4(config-router-af)#network 192.168.47.0 0.0.0.255
*Mar 18 11:47:58.047: %DUAL-5-NBRCHANGE: EIGRP-IPv4 2: Neighbor 192.168.47.7 (FastEthernet0/1) is up: new adjacency
//可以看到PE2和CE2的EIGRP邻居关系马上就恢复了；
```

并将EIGRP 1 的`ipv4 vrf A-Site2 autonomous-system 2`地址族重分发进MP-BGP的ipv4 vrf A-Site2地址族中；
```c
R4(config)#router bgp 1
R4(config-router)#address-family ipv4 vrf A-Site2
R4(config-router-af)#redistribute eigrp 2
```

### 验证与调试：

在VPN A站点1的C路由器R6上ping远程VPN A站点2的C路由器R8；
```c
R6(config-router)#do p 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 132/162/232 ms
//成功！
```

查看C路由器R8的IP路由表；
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
D EX     5.5.5.5 [170/286720] via 192.168.78.7, 00:10:06, FastEthernet0/0
        6.0.0.0/32 is subnetted, 1 subnets
D EX     6.6.6.6 [170/286720] via 192.168.78.7, 00:10:06, FastEthernet0/0
        7.0.0.0/32 is subnetted, 1 subnets
D        7.7.7.7 [90/156160] via 192.168.78.7, 00:14:15, FastEthernet0/0
        8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback0
D EX  192.168.15.0/24 [170/286720] via 192.168.78.7, 00:10:06, FastEthernet0/0
D     192.168.47.0/24 [90/30720] via 192.168.78.7, 00:14:15, FastEthernet0/0
D EX  192.168.56.0/24 [170/286720] via 192.168.78.7, 00:10:06, FastEthernet0/0
        192.168.78.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.78.0/24 is directly connected, FastEthernet0/0
L        192.168.78.8/32 is directly connected, FastEthernet0/0
//来自VPN A站点1的EIGRP路由变为EIGRP外部路由；
```

此时再来查看R1的IP BGP VPNv4路由表中关于前缀8.8.8.8/32的详细信息；
```c
R1#sh ip bgp vpnv4 vrf A-Site1 8.8.8.8
BGP routing table entry for 1:1:8.8.8.8/32, version 164
Paths: (1 available, best #1, table A-Site1)
    Not advertised to any peer
    Local, imported path from 1:2:8.8.8.8/32
    4.4.4.4 (metric 4) from 4.4.4.4 (4.4.4.4)
        Origin incomplete, metric 158720, localpref 100, valid, internal, best
        Extended Community: RT:1:2 Cost:pre-bestpath:128:158720 0x8800:32768:0 
        0x8801:2:133120 0x8802:65282:25600 0x8803:65281:1500
        0x8806:0:134744072
        mpls labels in/out nolabel/27
```
即便现在R6上所有来自VPN A站点2的EIGRP路由都变为了EIGRP外部路由，但PE1（R1）收到的VPNv4路由则不会携带拓展团体属性类型0x8804和0x8805字段；因为R6上的这些EIGRP外部路由是由于EIGRP AS号不同而造成的，而这些路由在远程VPN A站点2中并不是EIGRP外部路由。
