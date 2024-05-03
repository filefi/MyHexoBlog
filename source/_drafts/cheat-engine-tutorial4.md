---
title: Cheat Engine Tutorial 学习笔记4：指令查找
date: 2024-04-30 11:41:21
tags: [PWN, Reverse, CheatEngine]
categories: Reverse

---



本文重点介绍：

- 如何查找对特定地址进行访问或修改的操作指令

CE版本如下：

- Cheat Engine Version: 7.4和7.5

- Tutorial Version: v3.6



<!-- more -->

### Step 5

```text
Step 5: Code finder (PW=888899)
Sometimes the location of a value is stored at changes, when you restart the game, or even while you're playing. In that case you can use 2 things to still make a table that works.
In this step I'll try to describe how to use the Code Finder function.

The value down here will be at a different location each time you start the tutorial, so a normal entry in the address list wouldn't work.
First try to find the address. (You've got to this point so I assume you know how to do that.)
When you've found the address, right-click the address in Cheat Engine and choose "Find out what writes to this address". A window will pop up with an empty list.
Then click on the Change value button in this tutorial, and go back to Cheat Engine. If everything went right, there should be an address with assembler code there now.
Click it and choose the Replace option to replace it with code that does nothing. That will also add the code address to the code list in the Advanced Options window. (Which gets saved if you save your table.)

Click on Stop, so the game will start running normal again, and click on Close to close the window.
Now, click on Change value, and if everything went right the Next button should become enabled.

Note: When you're freezing the address with a high enough speed it may happen that Next becomes visible anyhow.
```

Step 5 主要介绍了如何找到有哪些汇编操作指令在修改特定地址中的值，并使用空指令`nop`替换该指令。

首先，使用精确值扫描找到 Health 的地址：

![](image-20240503184856179.png)

这里省略了 Health 地址的查找过程：

![](image-20240503185455807.png)

**找到 Health 的地址之后，接下来，我们要使用 CE 的 code finder 查找是哪些汇编操作指令在访问或修改 Health 所在地址中保存的值。**

鼠标右键点击 addresslist 中的地址条目，然后在右键菜单中点击`Find out what accesses this address`或者`Find out what writes to this address`，查找对该地址进行访问或修改的汇编操作指令。在这里，由于我们已经知道操作是在修改 Health 的值，那么基本上是在对 Health 所在地址进行写入操作，我们点击`Find out what writes to this address`：

![](image-20240503190432364.png)

点击`Find out what writes to this address`之后，会弹出提示框询问是否将 CE 的 debugger 附加到 CE Tutorial 的进程以便对 CE Tutorial 进行调试。点击 Yes 以继续：

![](image-20240503191226920.png)

然后，回到 Tutorial 点击 `Change value`改变 Health 的值。

![](image-20240503192355706.png)

点击`Change value`后，可以看到 debugger 跟踪到了一个向 Health 所在地址 `015A7E00` 写入值的操作指令。有时候，也有可能会跟踪到多个修改地址中值的操作指令，尤其是使用`Find out what accesses this address`时，可能会有很多操作指令都在访问某个特定地址。

鼠标双击找到的操作指令条目，以查看详情：

![](image-20240503194159974.png)

通过详情我们可以发现，对 Health 地址中的值进行实际修改的操作指令是：

```asm
mov [rax],edx
```

同时，rax 寄存器中的值确实就是 Health 所在的地址`015A7E00`，并且这也意味着这条指令通过一个指针找到了 Health 所在的地址`015A7E00`；那么，这个指针的值就应该是`015A7E00`。接下来的任务就是找到这个指针。

> 在C语言中，指针作为一种特殊的数据类型，其特点是：指针的值是一个内存地址，通过使用指针，我们可以直接访问或修改指针指向的地址的值。

**既然我们已经知道了这个指针的值`015A7E00`，就可以利用精确值查找的方式找到这个指针的地址。**

右键单击或者`Ctrl+C`可以方便地直接复制这个CE猜测的可能的地址值：

![](image-20240503195505674.png)

使用精确值扫描查找指针的值：

![](image-20240503200443215.png)

> 注意：对于32位平台，指针的大小为4 Bytes；64位平台指针的大小位8 Bytes。

通过搜索，我们可以到有哪些地址存放了`015A7E00`这个地址值。可以看到，地址`015ECA30`存放了`015A7E00`这个地址值，那么这地址可能就是我们要找的指针。

![](image-20240503200906802.png)

双击将其加入 addresslist：

![](image-20240503201219428.png)

双击这个条目，将这个地址修改为指针：

![i](image-20240503202036485.png)

但是，当前找到的这个指针并不是基于基地址的指针，也就是说，当CE Tutorial程序重新运行后，这个指针的地址会改变。所以我们还需要进一步查找是哪个操作指令在访问这个指针的地址：

![](image-20240503202909088.png)

为了找出哪条操作指令在通过这个指针修改Health的值，我们需要查找是哪条指令在访问这个指针，或者是哪条指令在访问这个指针指向的地址。在这里，我们先尝试`Find out what accesses this pointer`，验证是否有指令在访问这个指针：

![](image-20240503203834766.png)

点击`Change value`修改 Health 的值，以便debugger跟踪访问这个指针的操作指令：

![](image-20240503204851363.png)

可以看到，Health 的值已经改变了，但是并没有跟踪到有操作指令在访问这个指针：

![](image-20240503205117749.png)

我们先尝试`Find out what accesses the address pointed at by this pointer`，验证是否有指令通过这个指针字在访问这个指针所指向的地址：

![](image-20240503205215490.png)

点击`Change value`修改 Health 的值，以便debugger跟踪有哪些操作指令通过这个指针字在访问这个指针所指向的地址：

![](image-20240503205410666.png)

可以看到，有3条操作指令在通过这个指针访问其所指向的地址：

![](image-20240503205623963.png)

![](image-20240503205919362.png)
