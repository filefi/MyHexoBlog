---
title: Cheat Engine Tutorial 学习笔记2：指令查找
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

点击`Option`按钮，这将会用空指令`nop`替换现有指令`mov [rax],edx`：

![](image-20240503213822275.png)

这将会导致，当我们点击`Change value`时，在执行地址为`10002CB88`的指令时，不再执行`mov [rax],edx`而是执行空操作指令`nop`，即什么都不做，因此 Health 的值将不会被修改。同时，还会自动把这条指令的地址添加到 `Advanced Options` 中的 code list 中。

点击 `Advanced Options`打开 code list：

![](image-20240503215159161.png)

![](image-20240503215246295.png)

右键选中条目，打开右键菜单：

![](image-20240503215532647.png)

选择`Open disassembler at this location`，在此位置打开反汇编器：

![](image-20240503215732870.png)

点击`Change value`验证结果：

![](image-20240503214544723.png)

