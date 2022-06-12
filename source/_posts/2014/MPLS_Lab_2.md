---
title: MPLS 实验2：MPLS LDP会话保护
date: 2014-03-14 22:23:41
updated: 2014-03-14 22:23:41
tags: [MPLS, Network]
categories: Network
---


# 理论：MPLS LDP会话保护
在对两台直接连接LSR之间的LDP会话实施保护后，将会在这两台LSR之间建立基于目标的LDP会话。当这两台LSR之间的直连链路断开以后，只要这两台LSR之间存在可替代路径，基于目标的LDP会话将会得到维持。
    
# 实验环境：
- 模拟器：GNS3 0.8.6
- Cisco IOS：c7200-adventerprisek9-mz.151-4.M2.image


# 实验拓扑：
![topology](topo.png)

<!-- more -->

# 基本预配置：
## R1：
```
hostname R1
!
ip cef
!
multilink bundle-name authenticated
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
    ip address 192.168.14.1 255.255.255.0
    ip ospf 1 area 0
    no shut
mpls ip
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
    no shut
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.23.2 255.255.255.0
    ip ospf 1 area 0
    no shut
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
    no shut
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.23.3 255.255.255.0
    ip ospf 1 area 0
    no shut
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
    no shut
mpls ip
!
interface FastEthernet0/1
    ip address 192.168.14.4 255.255.255.0
    ip ospf 1 area 0
    no shut
mpls ip
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


# 实验与调试：
## 实验1：两台LSR相互将对方配置为受MPLS LDP会话保护的对等体；

配置ACL来指定需要被保护的LDP对等体；
```c
R1(config)#ip access-list standard PROTECT_R4
R1(config-std-nacl)#permit host 4.4.4.4
R1(config-std-nacl)#exit
```

配置MPLS LDP会话保护，并将其与指定需要保护的LDP对等体ACL相关联；
```c
R1(config)#mpls ldp session protection for PROTECT_R4 duration ?
    <30-2147483>  Holdup time in seconds
    infinite      Protect session forever after loss of link discovery
//选项duration指出持续时间，持续时间指的是在LDP链路的邻接关系断掉以后，需要得到保护（基于目标的LDP会话）的周期时间；

R1(config)#mpls ldp session protection for PROTECT_R4
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
            R1(config)#do sh mpls ldp nei 4.4.4.4 detail            
                Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
                    TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
                    Password: not required, none, in use
                    State: Oper; Msgs sent/rcvd: 61/59; Downstream; Last TIB rev sent 16
                    Up time: 00:42:56; UID: 2; Peer Id 1;
                    LDP discovery sources:
                      FastEthernet0/1; Src IP addr: 192.168.14.4 
                        holdtime: 15000 ms, hello interval: 5000 ms
                    Addresses bound to peer LDP Ident:
                      4.4.4.4         192.168.34.4    192.168.14.4    
                    Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
                    Clients: Dir Adj Client
                    LDP Session Protection enabled, state: Incomplete //可以看到针对邻居4.4.4.4,LDP会话保护已经启用；但因R1端还未配置，所以状态为Incomplete；
                        acl: PROTECT_R4, duration: 86400 seconds //持续保护时间为86400秒，即24小时；
                    Capabilities Sent:
                      [ICCP (type 0x0405) MajVer 1 MinVer 0]
                      [Dynamic Announcement (0x0506)]
                      [Typed Wildcard (0x050B)]
                    Capabilities Received:
                      [ICCP (type 0x0405) MajVer 1 MinVer 0]
                      [Dynamic Announcement (0x0506)]
                      [Typed Wildcard (0x050B)]
```
注意：根据《MPLS Fundamental》86页所述，默认的持续时间（Duration）为无限期，即永远，
但是经过试验发现，如果不指明选项duration，则持续保护时间为86400秒，即24小时；

而明确指定 duration 为 infinite（无限期）之后，持续保护时间才真正变为了 infinite（无限期）；
```c
R1(config)#mpls ldp session protection for PROTECT_R4 duration infinite 
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
R1(config)#do sh mpls ldp nei 4.4.4.4 detail                           
    Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 70/69; Downstream; Last TIB rev sent 16
        Up time: 00:51:05; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.4 
            holdtime: 15000 ms, hello interval: 5000 ms
        Addresses bound to peer LDP Ident:
            4.4.4.4         192.168.34.4    192.168.14.4    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Incomplete  //可以看到针对邻居4.4.4.4,LDP会话保护已经启用；但因R1端还未配置，所以状态为Incomplete；
            acl: PROTECT_R4, duration: infinite//持续保护时间为无限期，即永远保护；
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```
而当指定了duration为无限期（infinite）之后，持续保护时间才真正变为了infinite（无限期）；


验证：查看R4的MPLS LDP邻居列表中邻居R1的详细信息；
```c
R4(config)#do sh mpls ldp nei 1.1.1.1 detail
    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
        TCP connection: 1.1.1.1.646 - 4.4.4.4.40199
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 85/86; Downstream; Last TIB rev sent 16
        Up time: 01:05:13; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.1 
            holdtime: 15000 ms, hello interval: 5000 ms
        Addresses bound to peer LDP Ident:
            192.168.12.1    192.168.14.1    1.1.1.1         
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```
注意到，虽然此时R1上已经配置了针对邻居R4的LDP会话保护，但是由于R4上未做配置，所以在R4看来，仿佛这一切的一切并未发生过一般；
            
            
配置ACL来指定需要被保护的LDP对等体；
```c
R4(config)#ip access-list standard PROTECT_R1
R4(config-std-nacl)#permit host 1.1.1.1
R4(config-std-nacl)#exit
```

配置MPLS LDP会话保护，并将其与指定需要保护的LDP对等体ACL相关联；
```c
R4(config)#mpls ldp session protection for PROTECT_R1 duration infinite
```
            
验证：查看R4的MPLS LDP邻居列表中邻居R1的详细信息；
```c
R4(config)#do sh mpls ldp nei 1.1.1.1 detail                           
    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
        TCP connection: 1.1.1.1.646 - 4.4.4.4.40199
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 101/103; Downstream; Last TIB rev sent 16
        Up time: 01:19:22; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.1 
            holdtime: 15000 ms, hello interval: 5000 ms
            Targeted Hello 4.4.4.4 -> 1.1.1.1, active, passive;//基于目标的远程LDP会话已经建立；
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            192.168.12.1    192.168.14.1    1.1.1.1         
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Ready //可以看到R4针对邻居R1的LDP会话保护已经开启，并且状态为Ready，表示已经做好会话保护准备；
            acl: PROTECT_R1, duration: infinite//持续保护时间为无限期，即永远保护；
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
R1(config)#do sh mpls ldp nei 4.4.4.4 detail
    Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 108/106; Downstream; Last TIB rev sent 16
        Up time: 01:24:16; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.4 
            holdtime: 15000 ms, hello interval: 5000 ms
            Targeted Hello 1.1.1.1 -> 4.4.4.4, active, passive;//基于目标的远程LDP会话已经建立；
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            4.4.4.4         192.168.34.4    192.168.14.4    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Ready//可以看到R4针对邻居R1的LDP会话保护已经开启，并且状态为Ready，表示已经做好会话保护准备；
            acl: PROTECT_R4, duration: infinite//持续保护时间为无限期，即永远保护；
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
    ```

调试：断开R1和R4之间的直连链路；
```c
//将R1上与R4相连的F0/1口shutdown；
R1(config)#int f0/1
R1(config-if)#shutdown
R1(config-if)#
*Mar 15 11:24:27.663: %LDP-5-SP: 4.4.4.4:0: session hold up initiated  //LDP会话保护机制已经开始初始化；
*Mar 15 11:24:27.675: %OSPF-5-ADJCHG: Process 1, Nbr 4.4.4.4 on FastEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
R1(config-if)#
*Mar 15 11:24:29.619: %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down
*Mar 15 11:24:30.619: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
R1(config)#do sh mpls ldp nei 4.4.4.4 detail
    Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 120/116; Downstream; Last TIB rev sent 18
        Up time: 01:32:26; UID: 2; Peer Id 1;
        LDP discovery sources:
            Targeted Hello 1.1.1.1 -> 4.4.4.4, active, passive;
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            4.4.4.4         192.168.34.4    192.168.14.4    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Protecting //LDP会话保护正在进行中，状态为Protecting；
            acl: PROTECT_R4, duration: infinite
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```
            
            
查看R1的LIB表；
```c
R1(config)#do show mpls ldp bindings
    lib entry: 1.1.1.1/32, rev 2
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: 16
        remote binding: lsr: 4.4.4.4:0, label: 16
    lib entry: 2.2.2.2/32, rev 8
        local binding:  label: 16
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 17
    lib entry: 3.3.3.3/32, rev 13
        local binding:  label: 18
        remote binding: lsr: 2.2.2.2:0, label: 18
        remote binding: lsr: 4.4.4.4:0, label: 18
    lib entry: 4.4.4.4/32, rev 16
        local binding:  label: 20
        remote binding: lsr: 2.2.2.2:0, label: 20
        remote binding: lsr: 4.4.4.4:0, label: imp-null
    lib entry: 192.168.12.0/24, rev 4
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 19
    lib entry: 192.168.14.0/24, rev 18
        local binding:  label: 21
        remote binding: lsr: 2.2.2.2:0, label: 17
        remote binding: lsr: 4.4.4.4:0, label: imp-null
    lib entry: 192.168.23.0/24, rev 10
        local binding:  label: 17
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 20
    lib entry: 192.168.34.0/24, rev 14
        local binding:  label: 19
        remote binding: lsr: 2.2.2.2:0, label: 19
        remote binding: lsr: 4.4.4.4:0, label: imp-null

R1(config)#do show mpls ip binding
    1.1.1.1/32 
        in label:     imp-null  
        out label:    16        lsr: 2.2.2.2:0       
        out label:    16        lsr: 4.4.4.4:0       
    2.2.2.2/32 
        in label:     16        
        out label:    imp-null  lsr: 2.2.2.2:0        inuse
        out label:    17        lsr: 4.4.4.4:0       
    3.3.3.3/32 
        in label:     18        
        out label:    18        lsr: 2.2.2.2:0        inuse
        out label:    18        lsr: 4.4.4.4:0       
    4.4.4.4/32 
        in label:     20        
        out label:    20        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0       
    192.168.12.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 2.2.2.2:0       
        out label:    19        lsr: 4.4.4.4:0       
    192.168.14.0/24 
        in label:     21        
        out label:    17        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0       
    192.168.23.0/24 
        in label:     17        
        out label:    imp-null  lsr: 2.2.2.2:0        inuse
        out label:    20        lsr: 4.4.4.4:0       
    192.168.34.0/24 
        in label:     19        
        out label:    19        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0  
//可以看到，即便R1与R4的直连链路已经断开，但是R1的LIB表中仍然保留着通告自R4的标签绑定信息；
```
            
验证：查看R4的MPLS LDP邻居列表中邻居R1的详细信息；
```c
R4(config)#do sh mpls ldp nei 1.1.1.1 detail
    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
        TCP connection: 1.1.1.1.646 - 4.4.4.4.40199
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 117/121; Downstream; Last TIB rev sent 16
        Up time: 01:33:22; UID: 2; Peer Id 1;
        LDP discovery sources:
            Targeted Hello 4.4.4.4 -> 1.1.1.1, active, passive;
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            192.168.12.1    1.1.1.1         
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Protecting //LDP会话保护正在进行中，状态为Protecting；
            acl: PROTECT_R1, duration: infinite
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```

查看R4到LIB表；
```c
R4(config)#do show mpls ldp bindings
    lib entry: 1.1.1.1/32, rev 2
        local binding:  label: 16
        remote binding: lsr: 3.3.3.3:0, label: 17
        remote binding: lsr: 1.1.1.1:0, label: imp-null
    lib entry: 2.2.2.2/32, rev 4
        local binding:  label: 17
        remote binding: lsr: 3.3.3.3:0, label: 16
        remote binding: lsr: 1.1.1.1:0, label: 16
    lib entry: 3.3.3.3/32, rev 6
        local binding:  label: 18
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 18
    lib entry: 4.4.4.4/32, rev 8
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 20
        remote binding: lsr: 1.1.1.1:0, label: 20
    lib entry: 192.168.12.0/24, rev 10
        local binding:  label: 19
        remote binding: lsr: 3.3.3.3:0, label: 19
        remote binding: lsr: 1.1.1.1:0, label: imp-null
    lib entry: 192.168.14.0/24, rev 12
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 18
        remote binding: lsr: 1.1.1.1:0, label: 21
    lib entry: 192.168.23.0/24, rev 14
        local binding:  label: 20
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 17
    lib entry: 192.168.34.0/24, rev 16
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 19

R4(config)#do show mpls ip binding
    1.1.1.1/32 
        in label:     16        
        out label:    17        lsr: 3.3.3.3:0        inuse
        out label:    imp-null  lsr: 1.1.1.1:0       
    2.2.2.2/32 
        in label:     17        
        out label:    16        lsr: 3.3.3.3:0        inuse
        out label:    16        lsr: 1.1.1.1:0       
    3.3.3.3/32 
        in label:     18        
        out label:    imp-null  lsr: 3.3.3.3:0        inuse
        out label:    18        lsr: 1.1.1.1:0       
    4.4.4.4/32 
        in label:     imp-null  
        out label:    20        lsr: 3.3.3.3:0       
        out label:    20        lsr: 1.1.1.1:0       
    192.168.12.0/24 
        in label:     19        
        out label:    19        lsr: 3.3.3.3:0        inuse
        out label:    imp-null  lsr: 1.1.1.1:0       
    192.168.14.0/24 
        in label:     imp-null  
        out label:    18        lsr: 3.3.3.3:0       
        out label:    21        lsr: 1.1.1.1:0       
    192.168.23.0/24 
        in label:     20        
        out label:    imp-null  lsr: 3.3.3.3:0        inuse
        out label:    17        lsr: 1.1.1.1:0       
    192.168.34.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 3.3.3.3:0       
        out label:    19        lsr: 1.1.1.1:0 
//可以看到，即便R1与R4的直连链路已经断开，但是R4的LIB表中仍然保留着通告自R1的标签绑定信息；
```

调试：恢复R1和R4之间的链路连接；
```c
R1(config)#int f0/1
R1(config-if)#no shut
R1(config-if)#
*Mar 15 11:44:21.407: %LDP-5-SP: 4.4.4.4:0: session recovery succeeded //R1与R4的会话恢复成功；
R1(config-if)#
*Mar 15 11:44:23.279: %LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to up
R1(config-if)#
*Mar 15 11:44:24.279: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
```
    
    
## 实验2：两台LSR中只有一台支持LDP会话保护特性

要让 LDP 会话保护特性正常工作，你需要在两端的 LSR 上都启用该特性。如果其中一台 LSR 无法实现该特性的话，可以在另一台LSR上启用该特性，而无法启用 LDP 会话保护的 LSR 上需要通过配置命令 `mpls ldp discovery targeted-hello accept` 来接受基于目标的 LDP Discovery Hello报文。 
        
配置ACL指定接受其远程LDP会话LSR；
```c
R4(config)#ip access-list standard ACCEPT_R1_REMOTE
R4(config-std-nacl)#permit host 1.1.1.1
R4(config-std-nacl)#exit
```

配置R4接受来自ACCEPT_R1_REMOTE的远程LDP会话；
```c
R4(config)#mpls ldp discovery targeted-hello accept from ACCEPT_R1_REMOTE    
```

验证：查看R4的MPLS LDP邻居列表中邻居R1的详细信息；
```c
R4(config)#do sh mpls ldp nei 1.1.1.1 detail                              
    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
        TCP connection: 1.1.1.1.646 - 4.4.4.4.40199
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 141/146; Downstream; Last TIB rev sent 16
        Up time: 01:52:47; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.1 
            holdtime: 15000 ms, hello interval: 5000 ms
            Targeted Hello 4.4.4.4 -> 1.1.1.1, passive; //R4被动接受来自R1的远程LDP会话；关键字passive表示R4是被动方，被动接受与R1的远程LDP会话关系：
            holdtime: 90000 ms, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            192.168.12.1    1.1.1.1         192.168.14.1    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
R1(config-if)#do show mpls ldp neighbor 4.4.4.4 detail
    Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 146/140; Downstream; Last TIB rev sent 20
        Up time: 01:52:35; UID: 2; Peer Id 1;
        LDP discovery sources:
            FastEthernet0/1; Src IP addr: 192.168.14.4 
            holdtime: 15000 ms, hello interval: 5000 ms
            Targeted Hello 1.1.1.1 -> 4.4.4.4, active, passive; //R1与R4已建立远程LDP会话；关键字active和passive表示R1是远程会话的主动发起方和被动接受方；
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            4.4.4.4         192.168.34.4    192.168.14.4    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Ready //R1上针对邻居R4的LDP会话保护已经启用；并且状态为Ready，表示已经做了会话保护准备；
            acl: PROTECT_R4, duration: infinite
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
//可以看到，即使R4没有启用LDP会话保护特性，但只要R4接受来自R1基于目标的远程LDP会话报文，R1的LDP会话保护特性就可以正常运作；
```

调试：断开R1与R4之间的链路连接；
```c
//在R1上，将连接到R4的F0/1口shutdown；
R1(config-if)#int f0/1
R1(config-if)#shutdown
R1(config-if)#
*Mar 15 12:10:34.939: %LDP-5-SP: 4.4.4.4:0: session hold up initiated //R1上针对R4的LDP会话保护机制开始初始化；
*Mar 15 12:10:34.951: %OSPF-5-ADJCHG: Process 1, Nbr 4.4.4.4 on FastEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
R1(config-if)#
*Mar 15 12:10:36.899: %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down
*Mar 15 12:10:37.899: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
```

验证：查看R1的MPLS LDP邻居列表中邻居R4的详细信息；
```c
R1(config-if)#do sh mpls ldp nei 4.4.4.4 detail  
    Peer LDP Ident: 4.4.4.4:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 4.4.4.4.40199 - 1.1.1.1.646
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 175/168; Downstream; Last TIB rev sent 22
        Up time: 02:15:35; UID: 2; Peer Id 1;
        LDP discovery sources:
            Targeted Hello 1.1.1.1 -> 4.4.4.4, active, passive;
            holdtime: infinite, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            4.4.4.4         192.168.34.4    192.168.14.4    
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Clients: Dir Adj Client
        LDP Session Protection enabled, state: Protecting //R1针对邻居R4的LDP会话保护状态变为Protecting；
            acl: PROTECT_R4, duration: infinite
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
```

查看R1的LIB表；
```c
R1(config-if)#do sh mpls ldp bindings
    lib entry: 1.1.1.1/32, rev 2
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: 16
        remote binding: lsr: 4.4.4.4:0, label: 16
    lib entry: 2.2.2.2/32, rev 8
        local binding:  label: 16
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 17
    lib entry: 3.3.3.3/32, rev 13
        local binding:  label: 18
        remote binding: lsr: 2.2.2.2:0, label: 18
        remote binding: lsr: 4.4.4.4:0, label: 18
    lib entry: 4.4.4.4/32, rev 16
        local binding:  label: 20
        remote binding: lsr: 2.2.2.2:0, label: 20
        remote binding: lsr: 4.4.4.4:0, label: imp-null
    lib entry: 192.168.12.0/24, rev 4
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 19
    lib entry: 192.168.14.0/24, rev 22
        local binding:  label: 21
        remote binding: lsr: 2.2.2.2:0, label: 17
        remote binding: lsr: 4.4.4.4:0, label: imp-null
    lib entry: 192.168.23.0/24, rev 10
        local binding:  label: 17
        remote binding: lsr: 2.2.2.2:0, label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 20
    lib entry: 192.168.34.0/24, rev 14
        local binding:  label: 19
        remote binding: lsr: 2.2.2.2:0, label: 19
        remote binding: lsr: 4.4.4.4:0, label: imp-null

R1(config-if)#do sh mpls ip binding
    1.1.1.1/32 
        in label:     imp-null  
        out label:    16        lsr: 2.2.2.2:0       
        out label:    16        lsr: 4.4.4.4:0       
    2.2.2.2/32 
        in label:     16        
        out label:    imp-null  lsr: 2.2.2.2:0        inuse
        out label:    17        lsr: 4.4.4.4:0       
    3.3.3.3/32 
        in label:     18        
        out label:    18        lsr: 2.2.2.2:0        inuse
        out label:    18        lsr: 4.4.4.4:0       
    4.4.4.4/32 
        in label:     20        
        out label:    20        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0       
    192.168.12.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 2.2.2.2:0       
        out label:    19        lsr: 4.4.4.4:0       
    192.168.14.0/24 
        in label:     21        
        out label:    17        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0       
    192.168.23.0/24 
        in label:     17        
        out label:    imp-null  lsr: 2.2.2.2:0        inuse
        out label:    20        lsr: 4.4.4.4:0       
    192.168.34.0/24 
        in label:     19        
        out label:    19        lsr: 2.2.2.2:0        inuse
        out label:    imp-null  lsr: 4.4.4.4:0   
//注意到，由于LDP会话保护机制，R1的LIB表中仍然保留着R4通告来的标签；
```

验证：查看R4的MPLS LDP邻居列表中邻居R1的详细信息；
```c
R4(config)#do show mpls ldp nei 1.1.1.1 detail
    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 4.4.4.4:0
        TCP connection: 1.1.1.1.646 - 4.4.4.4.40199
        Password: not required, none, in use
        State: Oper; Msgs sent/rcvd: 201/208; Downstream; Last TIB rev sent 16
        Up time: 02:45:13; UID: 2; Peer Id 1;
        LDP discovery sources:
            Targeted Hello 4.4.4.4 -> 1.1.1.1, passive;
            holdtime: 90000 ms, hello interval: 10000 ms
        Addresses bound to peer LDP Ident:
            192.168.12.1    1.1.1.1         
        Peer holdtime: 180000 ms; KA interval: 60000 ms; Peer state: estab
        Capabilities Sent:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
        Capabilities Received:
            [ICCP (type 0x0405) MajVer 1 MinVer 0]
            [Dynamic Announcement (0x0506)]
            [Typed Wildcard (0x050B)]
//虽然R4没有启用LDP会话保护，但是R4仍然和R1保持着基于目标的LDP远程会话；
```

查看R4的LIB表；
```c
R4(config)#do show mpls ldp bindings
    lib entry: 1.1.1.1/32, rev 2
        local binding:  label: 16
        remote binding: lsr: 3.3.3.3:0, label: 17
        remote binding: lsr: 1.1.1.1:0, label: imp-null
    lib entry: 2.2.2.2/32, rev 4
        local binding:  label: 17
        remote binding: lsr: 3.3.3.3:0, label: 16
        remote binding: lsr: 1.1.1.1:0, label: 16
    lib entry: 3.3.3.3/32, rev 6
        local binding:  label: 18
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 18
    lib entry: 4.4.4.4/32, rev 8
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 20
        remote binding: lsr: 1.1.1.1:0, label: 20
    lib entry: 192.168.12.0/24, rev 10
        local binding:  label: 19
        remote binding: lsr: 3.3.3.3:0, label: 19
        remote binding: lsr: 1.1.1.1:0, label: imp-null
    lib entry: 192.168.14.0/24, rev 12
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 18
        remote binding: lsr: 1.1.1.1:0, label: 21
    lib entry: 192.168.23.0/24, rev 14
        local binding:  label: 20
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 17
    lib entry: 192.168.34.0/24, rev 16
        local binding:  label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 19

R4(config)#do show mpls ip binding
    1.1.1.1/32 
        in label:     16        
        out label:    17        lsr: 3.3.3.3:0        inuse
        out label:    imp-null  lsr: 1.1.1.1:0       
    2.2.2.2/32 
        in label:     17        
        out label:    16        lsr: 3.3.3.3:0        inuse
        out label:    16        lsr: 1.1.1.1:0       
    3.3.3.3/32 
        in label:     18        
        out label:    imp-null  lsr: 3.3.3.3:0        inuse
        out label:    18        lsr: 1.1.1.1:0       
    4.4.4.4/32 
        in label:     imp-null  
        out label:    20        lsr: 3.3.3.3:0       
        out label:    20        lsr: 1.1.1.1:0       
    192.168.12.0/24 
        in label:     19        
        out label:    19        lsr: 3.3.3.3:0        inuse
        out label:    imp-null  lsr: 1.1.1.1:0       
    192.168.14.0/24 
        in label:     imp-null  
        out label:    18        lsr: 3.3.3.3:0       
        out label:    21        lsr: 1.1.1.1:0       
    192.168.23.0/24 
        in label:     20        
        out label:    imp-null  lsr: 3.3.3.3:0        inuse
        out label:    17        lsr: 1.1.1.1:0       
    192.168.34.0/24 
        in label:     imp-null  
        out label:    imp-null  lsr: 3.3.3.3:0       
        out label:    19        lsr: 1.1.1.1:0      
//注意到，虽然R4没有启用LDP会话保护机制，但是R4的LIB表中同样保留着R1通告来的标签；
```
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
