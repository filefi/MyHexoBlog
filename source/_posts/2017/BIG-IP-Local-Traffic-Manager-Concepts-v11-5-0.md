---
title: BIG-IP Local Traffic Manager Concepts v11.5.0
date: 2017-11-30 11:30:15
tags: [F5, loadbalance]
categories: F5
---

> 本文是 F5官方文档 [BIG-IP Local Traffic Manager Concepts v11.5.0](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-concepts-11-5-0.html) 的中文翻译版。初次于2017年11月30日发布在 [此repo](https://github.com/filefi/CN_BIG-IP_Local_Traffic_Manager_Concepts_v11.5.0)。

# 第1章 介绍本地流量管理器（Local Traffic Manager）
## 1.1 什么是BIG-IP本地流量管理器？
BIG-IP本地流量管理（Local Traffic Manager）控制流入或流出局域网LAN（包括内联网intranet）的网络流量。

出于智能地调整网络服务器上负载的目的，LTM的一个常用功能是它拦截和重定向入站网络流量的能力。但是，调整服务器负载并不是唯一的本地流量管理方式。

LTM包含了各种功能，例如，执行检查和转换报头和内容数据，管理基于SSL证书的认证，以及压缩HTTP响应。这样做，BIG-IP系统不仅会定向流量到适合的服务器资源，而且还通过执行Web服务器通常执行的任务来增强了网络安全，并释放了服务器资源。

> 注意： BIG-IP LTM是组成BIG-IP产品系列的几种产品之一。BIG-IP产品线中的所有产品都运行在强大的流量管理操作系统（Traffic Management Operating System）（通常称为TMOS）上。

<!-- more -->

## 1.2 连接（Connection）和会话（Session）的超时设置
LTM有一些可以被设置，以促进活动（active）连接管理的超时设置。只要连接仍然处于活动状态（active），系统就会通过跟踪连接表中的连接来显式地管理每个连接。连接表（Connection table）包含关于客户端（client-side）和服务器端（server-side）连接的状态信息，以及客户端与服务器端之间的关系的状态信息。

连接表中每个连接都消耗系统资源来维护连接表条目和检查连接状态。当LTM必须判断一个连接何时不再处于活动状态（active），然后撤销（retire）连接，以避免耗尽关键系统资源。如果连接表不断增长，并保持未检查状态（unchecked），如内存和处理器周期这样的资源将存在风险。

当使用会话保持（Session persistence）时，你也可以管理会话保持表中的条目的持续时间。


## 1.3 连接收割（Connection reaping）
以正常方式关闭和重置的连接会自动从连接表中撤销（retire）。然而，由于许多原因，大量连接通常保持空闲，而没有正常关闭。因此，一旦这些连接被判断为处于非活跃状态（inactive），LTM必须收割（reap）这些连接。收割（reaping）是撤销或回收原本处于空闲的连接的过程。

由于你可以在多个地方配置超时设置，所以有时可能有不止一个超时设置在影响同一个连接。理解这一点很重要。最佳的超时配置是：为了保留系统资源，在确定连接处于非活动状态（inactive）并应该被撤销（retire）之前，将空闲连接保持一段适当的时间（取决于具体应用）。


## 1.4 空闲超时选项
通过与处理连接的虚拟服务器（virtual server）相关联的模板（protocol profiles）或SNAT，空闲连接可以被超时。基于SNAT automap 或 VLAN group设置，没有被虚拟服务器（virtual server）管理的连接也可以被超时。

应用于一个连接的最短超时值是始终生效的。但是，在某些情况下，你可以想要修改此行为。

例如，你可能已经配置了一个用于处理长期连接（long-standing connections）的forwarding virtual server，并且这些连接可能会长时间处于空闲状态（例如SSH会话）。在这种情况下，你可以在相关的protocol profile（在这个例子中，SSH使用的是TCP）中配置一个很长的空闲超时值。但是，如果SNAT automap功能也被启用，则默认的300秒静态超时值仍然生效。

### 影响连接收割的空闲超时设置
以下是一个包含影响连接收割的空闲连接超时设置的对象列表。对于每种对象类型，表中列出了默认值，以及该值是否是用户可配置的。

配置对象类型 | 默认值（秒） | 用户可配置
---|---|---
Fast L4, Fast HTTP, TCP, SCTP profiles | 300 | 是
UDP profiles | 60 | 是
SNAT automap | 300 | 否
VLAN group | 300 | 否



## 1.5 其他超时设置
LTM包含其他2个超时设置，但这些设置不影响连接收割。这些设置出现在 OneConnect™ 和 会话保持（persistence profile）中。

OneConnect超时值控制着空闲的服务器端（server-side）连接可用于重用（re-use）的时长。也就是说，在服务器端（server-side）连接空闲了一段时间之后，这个超时值可能会导致系统关闭此服务器端（server-side）连接。在这种情况下，由于连接从未被主动使用，所以没有活动的（active）客户端（client-side）连接受到影响，并且系统会为新连接透明地选择或建立另一个服务器端（server-side）连接。OneConnect超时设置不需要与其他模板（profile）的超时设置相一致。

会话保持（Persistence）超时设置其实是一个会话（Session）的空闲超时设置，而不是单个连接（connection）。因此，会话保持超时设置通常应该被设置为稍大于适用的连接空闲超时设置，以此来允许会话继续，即使该会话中的连接已经过期。

### 不影响连接收割的空闲超时设置
LTM包括其他2个空闲超时设置，但这些设置不会影响连接收割。这些设置出现在 ***OneConnect*** 和 **persistence *profile*** 中。此表展示了这些设置的默认值以及设置是否是用户可配置的。

配置对象类型 | 默认值（秒） | 用户可配置
---|---|---
OneConnect™ profiles | disabled | 是
Cookie Hash, Destination Address Affinity, Hash, SIP, Source Address Affinity, and Universal persistence profiles | 180 | 是
MSRDP and SSL persistence profiles | 300 | 是


## 1.6 关于network map
BIG-IP Configuration utility 包括被称为network map的功能。***network map*** 显示一个本地流量对象的汇总，以及BIG-IP系统上的virtual server，pool和pool member的可视化示意图。对于本地流量汇总和network map，你都可以使用检索机制来自定义显示内容，该机制根据您指定的条件过滤要显示的内容。系统会以蓝色来高亮显示所有检索操作的匹配结果。

### 过滤机制
通过使用过滤框（filter bar）中的Type和Status列表，以及Search Box，你可以过滤network map功能的结果。有了Search box，你可以选择输入特定的字符串。图1.1 展示了Network Map界面中的过滤选项。

![image](https://support.f5.com/kb/global/manual_images/MAN-0377-06_v6/filter_screen.png)  
**图1：Network Map界面中的过滤选项**

当使用Search Box时，你可以指定在搜索操作中系统所使用的文本字符串。默认值是星号（*）。Status和Type字段的设置决定了搜索的范围。系统使用所指定的搜索字符串来过滤显示在屏幕上的结果。

例如，如果你限制搜索的内容为只包括IP地址包含10.10的不可用结点（Node），此操作将返回这些结点（Nodes），连同Pool的成员（Members），Pool本身，相关联的VS，以及你显式应用于VS的iRules。系统会按VS Name的字母顺序对结果排序。

系统支持搜索以IPv4和IPv6的地址格式搜索名字，IP地址，以及IP地址：端口组合。如果会像有星号通配符包围着字符串那样来处理字符串。例如，你指定了`10`，系统会像你输入了`*10*`那样来进行有效查找。当然你也可以明确地包含星号通配符。例如，你可以使用下列字搜索字符串:`10.10.10.*:80`,`10.10.*`和`*:80`。如果你明确地包含通配符，系统也会相应地处理该字符串。例如，如果你指定`10*`，则系统假设你希望搜索IP地址以10开头的对象。

> 提示：在浏览器运行变慢和停止处理之前，浏览器可以渲染数据的量是有限制的。映射大型配置可能会接近这些限制; 因此，内存限制可能会阻止系统生成整个配置的network map
。如果发生这种情况，系统会发布一个警告（alert），指出您可以使用Network Map汇总界面来确定配置的复杂程度。这可以给你关于所生成的network map大小的指示。您可以修改搜索条件以返回较少的结果，这样生成的network map就不会遇到这些限制了。

### 对象汇总
当你第一次打开Network Map界面，该界面将显示本地流量对象的汇总摘要。此汇总摘要包括使用搜索机制指定的对象类型，每种对象类型的数量，以及对于每种对象类型，不同状态对象的数量。

汇总摘要会显示以下对象类型的数据：
- Virtual Server
- Pools
- Pool members
- Nodes
- iRules

> 注意：本地流量摘要仅包含被VS引用的那些对象。例如，如果您已经在系统上配置了一个Pool，但没有引用该Pool的VS，则本地流量摘要不包括此Pool，及其成员或摘要中的关联的节点。

此图显示了汇总系统上本地流量对象的network map界面的示例。

![image](https://support.f5.com/kb/global/manual_images/MAN-0377-06_v6/i_net_map_summary.png)

**图2：本地流量汇总摘要**


### network map显示
Network Map 显示了在系统上定义的对象名称和状态的可视化层次结构，这些对象类型包括 VS，Pools，Pool Member，Nodes以及iRules。Network Map能够显示上下文中的所有对象，并从顶部的VS开始。在屏幕顶部的 Status，Type 和 Search 设置决定了 Network Map 包含的对象。

当你将光标放在 Network map 中的一个对象上时，系统将显示悬停文本，其中包含关于该对象的信息。当你把光标放在伴随对象的状态图标上时，系统将显示悬停文本，其中包含关于该对象状态信息，文本也将出现 在Pool 的 Properties 界面。

系统按字母表顺序对对象进行排序，然后以层次化结构的方式组织依赖对象。

由于 Network Map 在上下文中显示对象的方式，更新后的界面（screen）也会显示与这些对象相关的其他状态、类型和名字的对象。这是因为 Network Map 在显示对象时， 总是在上下文中显示那些依赖它们的对象，以及它们依赖的对象。

例如，如果你有一个可用的 VS 和一个可用的 Pool，以及2个 Pool Member（其中1个可用，1个离线），然后从 Status 列表中选择 Offline，这将导致系统在上下文中显示离线（offline）的 Pool Member，以及可用的 VS 和 可用的 Pool。这是因为可用的 VS 和可用的 Pool 依赖于离线的 Pool Member。