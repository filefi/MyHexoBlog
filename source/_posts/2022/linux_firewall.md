---
title: Linux Firewall 学习笔记
date: 2022-04-10 16:35:21
tags: [linux, iptables, firewalld, nftables]
categories: Linux
---

# iptables

## netfilter 和 iptables

- netfilter：内核态，即不以文件的形式存在的防火墙。
- iptables：用户态，在`/sbin/iptables`存在的防火墙。

## 防火墙的分类

按保护范围划分:

- 主机防火墙:服务范围为当前一台主机
- 网络防火墙:服务范围为防火墙一侧的局域网

## 4 Tables

- filter
- nat
- mangle
- raw
- security

## 5 Chains

- PREROUTING
- INPUT
- FORWARD
- OUTPUT
- POSTROUTING

<!-- more -->

## Targets
A  firewall rule specifies criteria for a packet and a target.  If the packet does not match, the next rule in the chain is examined; if it does match, then the next rule is specified by the value of the target, which can be the name of a user-defined chain, one of the  targets  described in *iptables-extensions(8)*, or one of the special values `ACCEPT`, `DROP` or `RETURN`. 

- `ACCEPT`  : means to let the packet through. 
-  `DROP` : means to drop the packet on the floor. 
-  `RETURN` : means stop traversing this chain and resume at the next rule in the previous (calling) chain.  If the end of a built-in chain is reached or a rule in  a  built-in  chain  with  target  `RETURN`  is matched, the target specified by the chain policy determines the fate of the packet.`RETURN`代表停止遍历当前链，然后返回之前的链（调用当前链的链），并继续匹配之前链中的下一条规则。如果到达了内建链的底部，或者某条内建链中目标（target）为`RETURN`的规则被匹配，那么这条内建链的默认策略 (policy)决定了当前数据包的归宿。



# firewalld



基于zone概念：

1. 源IP或源网段关联到zone；
2. 接口（网卡）关联到zone；
3. 没有被源IP（源网段）或接口关联的zone所匹配时，关联到default zone；**所以，default zone中不要轻易放置服务！**

## 查看默认zone

```bash
[root@localhost ~]# firewall-cmd --get-default-zone
public
```

## 设置默认zone

```bash
[root@localhost ~]# firewall-cmd --set-default-zone=home
success
[root@localhost ~]# firewall-cmd --get-default-zone
home
```

## 将源网段关联到zone

相关命令：

```bash
[root@localhost ~]# firewall-cmd --help | grep source
  --get-zone-of-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
                       Print name of the zone the source is bound to [P]
  --service=<service> --add-source-port=<portid>[-<portid>]/<protocol>
                       Add a new source port to service [P only]
  --service=<service> --remove-source-port=<portid>[-<portid>]/<protocol>
                       Remove a source port from service [P only]
  --service=<service> --query-source-port=<portid>[-<portid>]/<protocol>
                       Return whether the source port has been added for service [P only]
  --service=<service> --get-source-ports
                       List source ports of service [P only]
  --list-source-ports  List source ports added [P] [Z] [O]
  --add-source-port=<portid>[-<portid>]/<protocol>
                       Add the source port [P] [Z] [O] [T]
  --remove-source-port=<portid>[-<portid>]/<protocol>
                       Remove the source port [P] [Z] [O]
  --query-source-port=<portid>[-<portid>]/<protocol>
                       Return whether the source port has been added [P] [Z] [O]
                       sources in a zone [P] [Z] [T]
                       sources in a zone [P] [Z]
                       and sources has been enabled for a zone [P] [Z]
  --list-sources       List sources that are bound to a zone [P] [Z]
  --add-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
                       Bind the source to a zone [P] [Z]
  --change-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
                       Change zone the source is bound to [Z]
  --query-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
                       Query whether the source is bound to a zone [P] [Z]
  --remove-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
                       Remove binding of the source from a zone [P] [Z]
```

将源网段192.168.16.0/24关联到zone home：

```bash
[root@localhost ~]# firewall-cmd --permanent --add-source=192.168.16.0/24 --zone=home
success
```

查看配置没有生效，需要重新加载配置：
```bash
[root@localhost ~]# firewall-cmd --list-all --zone=home
home (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

重新加载配置：
```bash
[root@localhost ~]# firewall-cmd --reload
success
```

再次查看zone配置，发现配置已生效：
```bash
[root@localhost ~]# firewall-cmd --list-all --zone=home
home (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 192.168.16.0/24
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

查询某个网段是否被添加到某个zone：

```bash
[root@localhost ~]# firewall-cmd --query-source=192.168.16.0/24 --zone=home
yes
[root@localhost ~]# firewall-cmd --query-source=192.168.1.0/24 --zone=home
no
```

将某个zone中的网段切换到另一个zone中：

```bash
[root@localhost ~]# firewall-cmd --change-source=192.168.16.0/24 --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 192.168.16.0/24
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --query-source=192.168.16.0/24 --zone=public
yes
```

从一个zone中移除关联的网段：

```bash
[root@localhost ~]# firewall-cmd --remove-source=192.168.16.0/24 --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-all --zone=public
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --list-sources --zone=public

```

## 将接口（网卡）关联到zone

相关命令：

```bash
[root@localhost ~]# firewall-cmd -h | grep interface
  --get-default-zone   Print default zone for connections and interfaces
  --get-zone-of-interface=<interface>
                       Print name of the zone the interface is bound to [P]
  --add-forward        Enable forwarding of packets between interfaces and
  --remove-forward     Disable forwarding of packets between interfaces and
  --query-forward      Return whether forwarding of packets between interfaces
  --list-interfaces    List interfaces that are bound to a zone [P] [Z]
  --add-interface=<interface>
                       Bind the <interface> to a zone [P] [Z]
  --change-interface=<interface>
                       Change zone the <interface> is bound to [P] [Z]
  --query-interface=<interface>
                       Query whether <interface> is bound to a zone [P] [Z]
  --remove-interface=<interface>
                       Remove binding of <interface> from a zone [P] [Z]
```

查看zone所绑定的接口：

```bash
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --list-interfaces --zone=public
ens160
```

将接口添加进zone：

```bash
[root@localhost ~]# firewall-cmd --add-interface=ens160 --zone=public --permanent
Warning: ZONE_ALREADY_SET: 'ens160' already bound to 'public'
success
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --list-interfaces --zone=public
ens160
```

将接口从zone中删除：

```bash
[root@localhost ~]# firewall-cmd --remove-interface=ens160 --zone=public --permanent
The interface is under control of NetworkManager and already bound to the default zone
The interface is under control of NetworkManager, setting zone to default.
success
[root@localhost ~]# firewall-cmd --list-interface --zone=public

[root@localhost ~]# firewall-cmd --list-all --zone=public
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

注意：当将一个接口（网卡）从一个zone中移除时，NetworkManager服务会自动将其添加进default zone。下例中，将接口`ens160`从zone public中移除，但default zone正好就是zone public，所以NetworkManager又将接口`ens160`绑定回了default zone即 zone public：

```bash
[root@localhost ~]# firewall-cmd --get-default-zone
public
[root@localhost ~]# firewall-cmd --remove-interface=ens160 --zone=public --permanent
The interface is under control of NetworkManager and already bound to the default zone
The interface is under control of NetworkManager, setting zone to default.
success
```

## 基本规则

firewalld基本规则只能以白名单方式添加端口或服务。

### 添加服务

相关命令：

```bash
[root@localhost ~]# firewall-cmd -h | grep service
  --get-services       Print predefined services [P]
  --new-service=<service>
                       Add a new service [P only]
  --new-service-from-file=<filename> [--name=<service>]
                       Add a new service from file with optional name [P only]
  --delete-service=<service>
                       Delete an existing service [P only]
  --load-service-defaults=<service>
  --info-service=<service>
                       Print information about a service
  --path-service=<service>
                       Print file path of a service [P only]
  --service=<service> --set-description=<description>
                       Set new description to service [P only]
  --service=<service> --get-description
                       Print description for service [P only]
  --service=<service> --set-short=<description>
                       Set new short description to service [P only]
  --service=<service> --get-short
                       Print short description for service [P only]
  --service=<service> --add-port=<portid>[-<portid>]/<protocol>
                       Add a new port to service [P only]
  --service=<service> --remove-port=<portid>[-<portid>]/<protocol>
                       Remove a port from service [P only]
  --service=<service> --query-port=<portid>[-<portid>]/<protocol>
                       Return whether the port has been added for service [P only]
  --service=<service> --get-ports
                       List ports of service [P only]
  --service=<service> --add-protocol=<protocol>
                       Add a new protocol to service [P only]
  --service=<service> --remove-protocol=<protocol>
                       Remove a protocol from service [P only]
  --service=<service> --query-protocol=<protocol>
                       Return whether the protocol has been added for service [P only]
  --service=<service> --get-protocols
                       List protocols of service [P only]
  --service=<service> --add-source-port=<portid>[-<portid>]/<protocol>
                       Add a new source port to service [P only]
  --service=<service> --remove-source-port=<portid>[-<portid>]/<protocol>
                       Remove a source port from service [P only]
  --service=<service> --query-source-port=<portid>[-<portid>]/<protocol>
                       Return whether the source port has been added for service [P only]
  --service=<service> --get-source-ports
                       List source ports of service [P only]
  --service=<service> --add-helper=<helper>
                       Add a new helper to service [P only]
  --service=<service> --remove-helper=<helper>
                       Remove a helper from service [P only]
  --service=<service> --query-helper=<helper>
                       Return whether the helper has been added for service [P only]
  --service=<service> --get-service-helpers
                       List helpers of service [P only]
  --service=<service> --set-destination=<ipv>:<address>[/<mask>]
                       Set destination for ipv to address in service [P only]
  --service=<service> --remove-destination=<ipv>
                       Disable destination for ipv i service [P only]
  --service=<service> --query-destination=<ipv>:<address>[/<mask>]
                       Return whether destination ipv is set for service [P only]
  --service=<service> --get-destinations
                       List destinations in service [P only]
  --service=<service> --add-include=<service>
                       Add a new include to service [P only]
  --service=<service> --remove-include=<service>
                       Remove a include from service [P only]
  --service=<service> --query-include=<service>
                       Return whether the include has been added for service [P only]
  --service=<service> --get-includes
                       List includes of service [P only]
  --list-services      List services added [P] [Z]
  --add-service=<service>
                       Add a service [P] [Z] [O] [T]
  --remove-service=<service>
                       Remove a service [P] [Z] [O]
  --query-service=<service>
                       Return whether service has been added [P] [Z] [O]
```

添加服务到指定zone：

```bash
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --add-service=http --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

将http服务从zone public中移除：

```bash
[root@localhost ~]# firewall-cmd --permanent --remove-service=http --zone=public
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-services --zone=public
cockpit dhcpv6-client ssh
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```



### 添加端口

相关命令：

```bash
[root@localhost ~]# firewall-cmd -h | grep port
  --get-ipset-types    Print the supported ipset types
  --service=<service> --add-port=<portid>[-<portid>]/<protocol>
                       Add a new port to service [P only]
  --service=<service> --remove-port=<portid>[-<portid>]/<protocol>
                       Remove a port from service [P only]
  --service=<service> --query-port=<portid>[-<portid>]/<protocol>
                       Return whether the port has been added for service [P only]
  --service=<service> --get-ports
                       List ports of service [P only]
  --service=<service> --add-source-port=<portid>[-<portid>]/<protocol>
                       Add a new source port to service [P only]
  --service=<service> --remove-source-port=<portid>[-<portid>]/<protocol>
                       Remove a source port from service [P only]
  --service=<service> --query-source-port=<portid>[-<portid>]/<protocol>
                       Return whether the source port has been added for service [P only]
  --service=<service> --get-source-ports
                       List source ports of service [P only]
  --list-ports         List ports added [P] [Z] [O]
  --add-port=<portid>[-<portid>]/<protocol>
                       Add the port [P] [Z] [O] [T]
  --remove-port=<portid>[-<portid>]/<protocol>
                       Remove the port [P] [Z] [O]
  --query-port=<portid>[-<portid>]/<protocol>
                       Return whether the port has been added [P] [Z] [O]
  --list-source-ports  List source ports added [P] [Z] [O]
  --add-source-port=<portid>[-<portid>]/<protocol>
                       Add the source port [P] [Z] [O] [T]
  --remove-source-port=<portid>[-<portid>]/<protocol>
                       Remove the source port [P] [Z] [O]
  --query-source-port=<portid>[-<portid>]/<protocol>
                       Return whether the source port has been added [P] [Z] [O]
  --list-forward-ports List IPv4 forward ports added [P] [Z] [O]
  --add-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
                       Add the IPv4 forward port [P] [Z] [O] [T]
  --remove-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
                       Remove the IPv4 forward port [P] [Z] [O]
  --query-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
                       Return whether the IPv4 forward port has been added [P] [Z] [O]
  --helper=<helper> --add-port=<portid>[-<portid>]/<protocol>
                       Add a new port to helper [P only]
  --helper=<helper> --remove-port=<portid>[-<portid>]/<protocol>
                       Remove a port from helper [P only]
  --helper=<helper> --query-port=<portid>[-<portid>]/<protocol>
                       Return whether the port has been added for helper [P only]
  --helper=<helper> --get-ports
                       List ports of helper [P only]
```

添加tcp端口8888到zone public：

```bash
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@localhost ~]# firewall-cmd --permanent --add-port=8888/tcp --zone=public
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp 8888/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

将tcp端口8888从zone public中移除：

```bash
[root@localhost ~]# firewall-cmd --permanent --remove-port=8888/tcp --zone=public
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# firewall-cmd --list-ports --zone=public
3306/tcp 8001/tcp 8002/tcp 8080/tcp 8090/tcp
[root@localhost ~]# firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 3306/tcp 8090/tcp 8001/tcp 8002/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

## 富规则（Rich Rules）

使用`--add-rich-rule`参数添加富规则。

