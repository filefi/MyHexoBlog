---
title: BIG-IP System iRules Concepts v11.6
date: 2017-11-30 09:05:31
tags: [F5, loadbalance]
categories: F5
---

> 本文是 F5官方文档 [BIG-IP System iRules Concepts v11.6](https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-system-irules-concepts-11-6-0.html) 的中文翻译版。初次于2017年11月30日发布在 [此repo](https://github.com/filefi/CN_BIG-IP_System_iRules_Concepts_v11.6)。

#  1 iRules介绍
## 什么是iRule
iRule是BIG-IP本地流量管理器（LTM）中的一个强大而灵活的功能，你可以用它来管理你的网络流量。使用基于行业标准工具命令语言（Tcl）的语法，iRules功能不仅允许您根据报头数据（header data）选择Pools，还可以通过搜索您自定义的任何类型的内容数据来定向流量。因此，iRules功能显著地增强了您自定义内容交换（content switching）以适应您确切需求的能力。
> 重要：有关iRules语法的完整和详细信息，请参阅F5 Networks DevCentral网站 http://devcentral.f5.com。请注意，iRules必须符合标准的Tcl语法规则; 因此，有关Tcl语法的更多信息，请参见 http://tmml.sourceforge.net/doc/tcl/index.html。

如果您想要单独的连接来命中（target）除为Virtual Server定义的默认Pool之外的Pool，则你就可以写这样一个iRule脚本。iRules允许您更直接地指定要将流量定向到你想要的目的地。使用iRules，您不仅可以将流量发送到Pool，还可以向单个Pool Member，端口或URI发送流量。您创建的iRules可以很简单，也可以很复杂，具体取决于您的内容交换（content-switching）需求。

```tcl
when CLIENT_ACCEPTED {
    if { [IP::addr [IP::client_addr] equals 10.10.10.10] } {
        pool my_pool 
    } 
}
```

当客户端连接被接受时，这条iRule会被触发。这时，如果客户端的地址与 `10.10.10.10` 相匹配，将使得本地流量管理器（LTM）将数据包发送到my_pool池。

<!-- more -->

使用称为通用检查引擎（Universal Inspection Engine）的功能，您可以编写一个搜索数据包头部或实际数据包内容的iRule，然后根据该检索结果来定向数据包。iRules还可以根据客户端认证的结果来定向数据包。

iRules不仅可以将流量定向到特定Pool，还可以将流量定向到单个Pool Member，甚至包括特定端口号和URI路径，以实现会话保持（persistence）或满足特定的负载均衡要求。

用于编写iRules的语法是基于工具命令语言（Tcl）编程标准。因此，您可以使用许多标准Tcl命令，以及本地流量管理器（LTM）提供的强大的扩展集，以帮助您进一步提高负载均衡效率。

> 重要提示：引用iRule中的对象时，必须包含对象的完整路径名。

### iRule命令
iRule中的 ***iRule命令*** 会使本地流量管理器（LTM）采取一些动作，如查询数据，操作数据，或指定流量的目的地。您可以在iRules中包含的命令类型有：

#### 声明命令（Statement Commands）
这些命令会引起诸如选择流量目的地或分配SNAT转换地址等动作。语句命令的一个例子是`pool <name>`，它将流量引导到命名的负载平衡池（Pool）。

#### 查询或操纵数据的命令
一些命令会搜索报头（header）和内容数据，而其他命令执行数据操作，例如将header插入到HTTP请求中。查询命令的一个例子是 `IP:: remote_addr`，它搜索并返回连接的远程IP地址。数据操作命令的一个例子是`HTTP::header remove <name>`，它从请求或响应中删除命名的header的最后一次事件（occurrence）。

#### 工具命令（Utility Commands）
这些命令是对解析和操作内容非常有用的功能。一个工具命令的例子是`decode_uri <string>`，它使用HTTP URI编码解码命名的字符串并返回结果。

### 事件声明
iRules是事件驱动的，这意味着本地流量管理器(LTM)会根据您在iRule中指定的事件来触发iRule。***事件声明*** 是iRule中一个事件的详细描述，这个事件会导致本地流量管理器（LTM）不论何时，只要事件发生就触发iRule。事件`HTTP_REQUEST`是触发iRule事件声明的例子，每当系统收到HTTP请求时，此事件会触发iRule；还有事件` CLIENT_ACCCEPTED`，此事件会在客户端建立连接时触发iRule。

```tcl
when HTTP_REQUEST { 
    if { [HTTP::uri] contains "aol" } {
        pool aol_pool 
    } else { 
        pool all_pool 
    } 
}
```

### 运算符
iRule运算符会比较表达式中的两个操作数。

例如，您可以使用`contains`运算符将变量操作数与常量进行比较。您可以通过创建一个if语句来表示以下内容：“如果HTTP URI包含aol，发送到aol_pool池。



## 创建iRule
您创建一个iRule来自定义BIG-IP系统处理流量的方式。
1. 在主选项卡（Main Tab）上，以此点击**Local Traffic > iRules**.
2. 点击**Create**。
3. 在**Name**字段中，键入名称，如my_irule.iRule的完整路径名称不能超过255个字符.
4. 在**Definition**字段中，使用“工具命令语言（Tcl）”语法键入iRule的语法。 有关iRules语法的完整和详细信息，请参阅F5 Networks DevCentral网站 http://devcentral.f5.com.
5. 点击**Finished**.


---

# 2 iRule命令
## iRule命令类型
有三种iRule命令类型：
- 声明命令（Statement Commands）
- 查询与操作命令（Query and Manipulation Commands）
- 工具（Utility）命令（也称为函数）

### 声明命令
iRules中一些可用的命令被称为声明命令（Statement Commands）。***声明命令*** 使得LTM能够执行各种不同的动作。例如，其中一些命令可以指定要使用LTM来重定向的pool和server。其他命令指定实现SNAT连接的翻译地址（Translation Address）。还有其他命令用于指定类似data group或者persistence profile的对象。

> 有关语句命令的完整列表，请参阅F5 Networks DevCentral网站 http://devcentral.f5.com。

### 查询和操作命令
使用iRules命令，你可以查询包含在一个请求或相应的头部（Header）或内容中的特定数据，或者操作这些数据。数据处理指的是插入，替换和删除数据，以及设置在头部（Header）和Cookie中找到的某些值。

例如，使用iRules中的`IP::idle_timeout`命令，你可以查询当前被设置在数据包报头中的空闲超时（Idle Timeout）值，然后相对应地对数据包进行负载均衡。你也可以使用`IP :: idle_timeout`命令将空闲超时设置为您选择的特定值。

> 有关iRules命令的命名空间的完整列表，请参阅F5 Networks DevCentral网站 http://devcentral.f5.com。

### 工具命令
LTM包括一些你可以在iRules中使用的工具命令。你可以使用这些命令来解析和检索内容，将数据编码为ASCII格式，校验数据完整性，以及检索关于Active的pool和pool member的信息。


## pool命令
一旦你已经在iRules中指定了一个查询，你就可以使用`pool`命令来选择一个你希望LTM向其发送请求的负载均衡Pool。这是一个此命令的示例：

```tcl
when HTTP_REQUEST { 
    set uri [HTTP::uri] 
    if { $uri ends_with ".gif" } {
        pool my_pool 
    } elseif { $uri ends_with ".jpg" } {
        pool your_pool
    } 
}
```

## node命令
作为`pool`命令的替代命令，你也可以写一个定向流量到指定服务器（server）的iRule。要这样做，你可以使用`node`命令。

```tcl
when HTTP_REQUEST { 
    if { [HTTP::uri] ends_with ".gif" } {
        node 10.1.2.200 80 
    } 
}
```

## 选择缓存服务器池（pool）的命令
你可以创建一个iRule，这个iRule可以从缓存服务器池（pool）中选择一个服务器（Server）。这是一个从缓存服务器池中选择一个服务器的示例：

```tcl
when HTTP_REQUEST {
    # This line specifies the expressions that determine whether the BIG-IP system sends requests to the cache pool:
    if { [HTTP::uri] ends_with "html" or [HTTP::uri] ends_with "gif" } {
        pool cache_pool
        set key [crc32 [concat [domain [HTTP::host] 2] [HTTP::uri]]]
        set cache_mbr [persist lookup hash $key node]
        if { $cache_mbr ne "" } {
            # This line verifies that the request is not coming from the cache:
            if { [IP::addr [IP::remote_addr] equals $cache_mbr] } {
                # This line sends the request from the cache to the origin pool:
                pool origin_pool
                return
            }
        }
    #These lines ensure that the persistence record is added for this host/URI:
    persist hash $key
    } else {
        pool origin_pool
    }
}
```

> 注意：LTM是在BIG-IP系统接收到URI的请求时，重定向URI到一个新的缓存服务器，而不是在pool member变为不可用时进行这一重定向操作。


## `HTTP::redirect`命令
除了配置iRule来选择指定的pool，你也可以使用 `HTTP::redirect` iRule命令来重定向HTTP请求到特定位置（location）。这个位置（location）可以是一个主机名或是一个URI。

以下是一个被配置来重定向HTTP响应的iRule。
```tcl
when HTTP_RESPONSE {
    if { [HTTP::status] contains "404"} {
        HTTP::redirect "http://www.siterequest.com/" 
    }
}
```

这是一个重定向HTTP请求的iRule示例：
```tcl
when HTTP_REQUEST {
    if { [HTTP::uri] contains "secure"} { 
        HTTP::redirect "https://[HTTP::host][HTTP::uri]"
    } 
}
```

## `snat`和`snatpool`命令
iRules功能包括2个声明命令`snat`和`snatpool`。使用`snat`命令可以为iRule中的初始（Original）IP地址分配一个指定的翻译地址（Translation Address），而不是使用BIG-IP Configuration utility中的SNAT界面。

使用`snatpool`命令也可以为初始（Original）IP地址分配翻译地址（Translation Address），尽管不像`snat`命令，`snatpool`命令使得LTM从你之前创建的一个特定SNAT池（pool）中选择翻译地址。

---

# 3 iRule评估（evaluation）
## 关于iRule评估（evaluation）
在不存在iRule的基本系统配置中，LTM将入站流量引导到接收这些流量的VS的默认pool。然而，你可能想让LTM引导某些连接到其他目的地。如果想这样做，可以写一个引导流量到其他目的地的iRule，但这依赖于某种类型的事件发生。否则，流量继续被引导到给VS所分配的默认pool。

因此，每当一个你已经在iRule中指定的事件发生时，iRules都将被评估（evaluate）。例如，如果iRule包含事件声明`CLIENT_ACCEPTED`，那么每当LTM接受一个客户端连接，iRule将被触发。然后，LTM遵循iRule的剩余部分来决定数据包的目的地。

> 注意：当你对iRule进行持久化修改，如果连接表中已经存在连接，那么，修改只有在连接到期后才会生效。同理，当你启用iRule的日志，然后修改iRule（或者变更日志消息本身），也是类似的。


## 事件类型
iRule命令语法包括几种你可以在iRule中指定的事件声明类型。例如：
- 全局事件，例如`CLIENT_ACCEPTED`
- HTTP事件，例如`HTTP_REQUEST`
- SSL事件，例如`CLIENTSSL_HANDSAKE`
- 认证（Authentication）事件，例如`AUTH_SUCCESS`

> 有关iRule事件及其说明的完整列表，请参阅F5 Networks DevCentral网站 http://devcentral.f5.com。


## iRule上下文（context）
对于每个你在iRule中指定的事件，你也可以指定由关键字`clientside`或`serverside`所表示的上下文（context）。因为每个事件都有与之关联的默认上下文，所以，如果你想要修改默认的上下文（context），你只需要声明一个上下文（context）。

这个例子展示了`my_iRule1`，它包含事件声明 `CLIENT_ACCEPTED`，以及iRule命令`IP::remote_addr`。在这种情况下，iRule命令返回的IP地址是客户端的IP地址。因为事件声明`CLIENT_ACCEPED`的默认上下文（context）是`clientside`。

```tcl
when CLIENT_ACCEPTED { 
    if { [IP::addr [IP::remote_addr] equals 10.1.1.80] } {
        pool my_pool1 
    }
}
```

同样地，如果你在iRule中包括事件声明`SERVER_CONNECTED`，以及iRule命令`IP::remote_addr`，那么iRule命令返回的IP地址就是服务器的IP地址。因为，事件声明`SERVER_CONNECTED`的默认上下文是`serverside`。

上述例子展示了当你在处理iRule命令时，编写使用默认上下文（context）的iRule
将发生什么。然而，你可以显式地指定关键字`clientside`和`serverside`来变更iRule命令的行为。

继续之前的例子，下面的例子展示了事件声明`SERVER_CONNECTED`和显式地为iRule命令`IP::remote_addr`指定关键字`clientside`。在这种情况下，iRule命令返回的IP地址就是客户端的IP地址，尽管事件声明的默认上下文是服务器端。


```
when SERVER_CONNECTED {
    if { [IP::addr [IP::addr [clientside {IP::remote_addr}] equals 10.1.1.80] } {
        discard 
    } 
}
```

> 注意：你可以通过使用一个紧跟在事件名后的关键字`when`在iRule中进行事件声明。这展示了iRule中事件声明的一个例子。


## VS的iRules分配
当你将多个iRules作为资源分配给VS时，考虑这些iRules在VS中被列出的顺序是很重要的。这是因为LTM以可用的iRules被列出的顺序来处理重复的iRule事件。因此，一个iRule事件可以终结事件触发，从而阻止LTM触发随后的事件。

> 如果一个iRule引用了一个profile，LTM最后处理这种类型的iRule，而不管它本身在VS的iRules列表中的顺序。


---


# 4 iRules与管理分区
你应该了解与管理分区相关的某些iRule配置概念：
- iRule可以引用任何对象，无论所引用的对象属于哪个分区。例如，贮存（reside）在分区 *partition_a* 的iRule可以包含指定了贮存（reside）在分区 *partition_b* 的pool对象的pool语句。
- 你可以只从贮存（reside）在当前写入分区（Write Partition）或分区 *Common* 中的VS里删除iRule分配。
- 注意：你可以只将iRule与贮存（reside）在当前写分区（Write partition）或分区 *Common* 中的VS进行关联。
- 你可以将现有的iRule与多个VS进行关联。在这种情况下，iRule将成为与在当前写分区（Write partition）中的每个VS相关联的唯一iRule。因为这条命令覆盖了之前所有的iRule分配。F5不推荐使用此命令。

---

# 5 iRules与本地流量Profiles
## iRules与模板（Profiles）
当你在写iRule时，你可能想要iRule识别特定模板（profile）配置的值，以便能够做出更加有根据的流量管理决定。幸运的是，iRules功能包括这样一个命令，这个命令是专门设计来读取你在iRule脚本中指定的模板（profile）设置的值。

iRule不仅可以读取模板（profile）配置的值，还可以覆盖某些设置的值。这意味着你可以针对个别连接（connection）应用不同的配置值。这里的个别连接指的是所配置的值与LTM应用于大多数通过VS的连接的配置值所不同的连接。

## `profile`命令
iRules功能包含一个叫作`PROFILE`的命令。当你在iRule中指定`PROFILE`命令和指定模板（profile）类型和设置，iRule会读取指定模板（profile）的配置值。为此，iRule会找到分配给VS的指定模板（profile）类型，然后读取你在`PROFILE`中命令序列中指定的配置值。然后，iRule可以使用此信息来管理流量。

例如，你可以在你的iRule中指定命令`PROFILE::tcp idle_timeout`。然后，LTM会找到分配给VS的TCP profile（比如，`my_tcp`）,并查询你给空闲超时（Idel Timeout）设置所分配的值。

## 覆盖模板（profile）配置的命令
一些用于查询和造作报头和数据内容的iRule命令在各种profile中有着等价的配置。当你在iRule中使用这些命令，并且一个事件触发了这条iRule，LTM将使用在iRule中指定的值覆盖这些profile设置的值。

例如，HTTP profile可能指定某个用于压缩HTTP数据的缓冲区大小，但是你可能想要为特定类型的HTTP连接指定一个不同的缓冲区大小。在这种情况下，你可以在你的iRule中包含命令`HTTP::compress_buffer_size`，以指定一个不同于profile中的值。

---

# 6 数据组 - Data Groups
## 关于数据组
编写iRules时，数据组是非常有用的。一个数据组只是一组相关的元素，例如AOL客户端的一组IP地址。当您用`class match`命令或 运算符`contains`指定数据组时，您不需要在iRule表达式中列出多个值作为参数。

您可以定义三种类型的数据组：地址，整数和字符串。

BIG-IP系统包括三个预配置数据组： *private_net*，*images*，和 *aol*。

要了解数据组的有用性，首先要了解`class match`命令和运算符`contains`。

> 注意：您只能根据你的用户角色和分区访问分配来管理您有权限管理的那些数据组。

> 警告：不要试图修改或删除三种预配置数据组中的任何一种（private_net，images，和aol）。否则可能会产生不良后果。


### 关于`class match`命令
BIG-IP系统包括一个称为`class`的iRule命令 ，此命令具有一个`match`选项，根据iRule中使用的命令是否表示特定数据组的成员，您可以使用该命令来选择Pool。当你使用`class`命令，BIG-IP系统知道跟在标识符后面的字符串是数据组的名称。

例如，如果`IP::remote_addr`命令的值是数据组AOL的成员，使用`class`命令，您可以使BIG-IP系统将所有入站的AOL连接负载均衡到aol_pool池。在这种情况下，`class match`命令表示命名为aol的对象是一个值的集合（即一个数据组）。

```tcl
when CLIENT_ACCEPTED { 
    if { [class match [IP::remote_addr] equals aol] } {
        pool aol_pool 
    } else {
        pool all_pool
    } 
}
```

### 存储选项
使用LTM，你能够以2种方式存储数据，*内嵌存储（in-line storage）* 或者 *外部存储*。

#### 内嵌存储（in-line storage）
当你创建data group时，LTM会自动把所创建的data group完整地保存在配置文件`bigip.conf`中。这种存储类型称为

一般来说，由于对大数据组（data group）有大量检索要求，*内嵌存储（in-line storage）* 会使用额外的系统资源。因此，LTM向你提供了外部存储数据的能力，也就是，存储在`bigip.conf`文件以外的文件。

#### 外部存储（External storage）
你可以选择将数据组（data group）存储在BIG-IP系统上的其他位置，也就是说，`bigip.conf`以外的文件。这样的数据组（data group）称为*外部数据组（external data groups）*。因为数据组（data group）被存储在其他位置，`bigip.conf`文件本身只包含数据组（data group）的文件名和元数据（meta-data）。在外部存储的数据组（data group）文件中的数据以逗号分隔的值的列表（CSV格式）被存储。

> 重要：如果你尝试加载包含外部数据组（data group）元数据（meta-data）的`bigip.conf`文件，并且此文件是BIP-IP 系统v9.4之前的版本创建的，系统将产生错误。外部数据组的元数据（meta-data）包含关键字`extern`，也就是这个关键字导致了加载过程中错误的产生。BIP-IP 系统v9.4及其以后版本，`bigip.conf`文件已经不再需要关键字`extern`了。

为了创建外部数据组，你要先使用BIG-IP Configuration utility的 **System** 选项从其他位置导入一个文件。然后，使用**Local Traffic iRules**配置界面，来创建一个基于该导入文件的外部数据组。

根据平台硬件和可用内存（建议使用8GB或更多内存）外部数据组（external data group）扩展到超过10,000,000条条目。拥有更大数据项目的数据组可以用较少条目的外部数据来支持。此外，对外部数据组的更新是完全原子的（atomic）：例如，只有新数据成功完成加载后，系统才会更新数据组。你可以使用命令`[class exists xyz]`来检查数据组是否已经完成加载了。


### 关于data groups的文件导入
使用BIG-IP Configuration utility，你可以导入包含你想要在data group中使用的内容的外部文件。当你将一个现有文件导入BIG-IP系统，那么BIG-IP系统会创建一个包含指定内容（地址，字符串或整数）类型的data group。

#### 导入data group的文件
使用BIG-IP Configuration utility，你可以从外部系统导入文件，并使用此文件来创建data group。
1. 在主标签页，依次点击 **System > File Management > Data Group File List > Import**。
2. 对于文件名设置，点击 **Browse**。系统将打开一个浏览窗口，以便你可以定位到你希望导入到BIG-IP系统的文件。
3. 在**Name**字段中，为导入的文件输入一个新名字。
4. 从**File Content**列表中选择数据组的内容类型。
5. 在**Key/Value Pair Separator**字段中，保留默认值或删除值，然后指定一个新的分隔符。
6. 在**Data Group Name**字段中，输入data group的名字。
7. 点击**Import**按钮。

#### 查看已导入的data group文件的列表
使用BIG-IP Configuration utility，你可以查看已经导入到BIG-IP系统的data group文件的列表。
1. 在主界标签页，依次点击 **System > File Management > Data Group File List**。
2. 在名字列，查看文件的列表。

---

# 7 iFiles
## 关于iRules文件的导入
如果你想要写iRule，而这个iRule要引用一个贮存（reside）在其他系统上的文件，你必须先将此文件导入到BIG-IP系统里。然后，你可以将此文件转换为iRule可以引用的iFile。

### 为iRule导入文件
在你执行此操作之前，你想要导入的此文件必须贮存（reside）在你指定的系统上。

你可以将文件从其他系统导入到BIG-IP系统，这是编写引用该文件的iRule的第一步。
1. 在主标签页，依次点击 **System > File Management > iFile List > Import**。
2. 对于**File Name**的设置，点击 **Browse**。系统将打开一个浏览窗口，以便你可以定位到你希望导入到BIG-IP系统的文件。
3. 浏览文件，然后点击**Open**。你选择的文件的名字将出现在**File Name**设置中。
4. 在**Name**字段，输入新的文件名，例如*1k.html*。新的文件名将出现在导入的文件的列表中。
5. 点击**Import**按钮。

在你执行此任务后，你导入的文件将贮存（reside）在BIG-IP系统上。

### 查看已导入文件的列表
出于在iRule中引用这些文件的目的，你可以执行此操作来查看已经导入到BIG-IP系统的文件的列表。
1. 在主标签页，依次点击 **System > File Management > iFile List**。
2. 在名字列，查看iFiles的列表。
3. 点击**Cancel**按钮。


## 关于iFiles
使用BIG-IP Configuration utility，你可以创建被称为iFile的特殊文件。一个 *iFile*是一个文件，此文件基于你之前从其他系统导入到BIG-IP系统上的外部文件。基于所指定的一个iRule事件，你可以从iRule中引用一个iFile。

为了创建一个iFile，并在iRule中使用它，你要从主标签页的**Local Traffic**选项开始。

> 重要：在创建iFile之前，你必须将文件从其他系统导入到BIG-IP系统。

### 查看iFiles列表
出于在iRule中引用这些文件的目的，你可以执行此操作来查看已经导入到BIG-IP系统的文件的列表。
1. 在主标签页，依次点击 **Local Traffic > iRules > iFile List**。
2. 在名字列，查看你之前在BIG-IP上创建的iFiles的列表。

### 创建iFile
作为先决条件，确保当前管理分区被设置为你想要保存（reside）iFile的分区。还要确保该文件已经导入到BIG-IP系统了。

你可以执行此操作来创建你之后能够在iRule中引用的iFile。
1. 在主标签页，依次点击 **Local Traffic > iRules > iFile List**。
2. 点击**Create**。
3. 在**Name**字段中，输入新的iFile文件名，例如 *ifileURL*。
4. 从**File Name**列表中，选择导入的文件对象的名字，例如 *1k.html*。
5. 点击**Finished**。新的iFile就会出现在iFiles列表中。

这个操作的结果是，你现在有了一个iRule可以引用的文件。


## 引用iFile的iRule命令
有了这些iRule命令，你可以从iRule中引用新的iFile：
- `[ifile get IFILENAME]`
- `[ifile listall]`
- `[ifile attributes IFILENAME]`
- `[ifile size IFILENAME]`
- `[ifile last_updated_by IFILENAME]`
- `[ifile last_update_time IFILENAME]`
- `[ifile revision IFILENAME]`
- `[ifile checksum IFILENAME]`
- `array set [file attributes IFILENAME]`

这个简单的iRule展示这些命令中的其中一些：
```tcl
ltm rule ifile_rule { 
    when HTTP_RESPONSE { 
        # return a list of iFiles in all partitions 
        set listifiles [ifile listall]
        log local0. "list of ifiles: $listifiles"
        
        # return the attributes of an iFile specified 
        array set array_attributes [ifile attributes "/Common/ifileURL"]
        
        foreach {array attr} [array get array_attributes ] {
            log local0. "$array : $attr"
        } 
        
        # serve an iFile when http status is 404. 
        set file [ifile get "Common/ifileURL"] 
        log local0. "file: $ifile" 
        if { [HTTP::status] equals "404" } {
            HTTP::respond 200 ifile "/Common/ifileURL" 
        } 
    } 
}
```