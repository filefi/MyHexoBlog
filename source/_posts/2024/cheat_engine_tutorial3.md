---
title: Cheat Engine Tutorial 学习笔记3：手动查找指针与基地址指针
date: 2024-04-30 23:17:21
tags: [PWN, Reverse, CheatEngine]
categories: Reverse
---



本文重点介绍：
- 手动查找指针与基地址指针

CE版本如下：

- Cheat Engine Version: 7.4和7.5

- Tutorial Version: v3.6



<!-- more -->


# 指针与基地址指针

使用精确或模糊值扫描找到的值的地址，在每次重新运行程序后可能都是会变的。甚至，很多情况下，不需要重新运行程序，只是切换了程序或游戏的菜单，或是游戏结束了一轮战斗，或者游戏切换了地图，就足以让刚才找到值所在地址引用失效。

为了让外挂在每次重新运行程序后都能准确地定位到特定目标值所在的地址，就需要找到基于程序基地址指向特定目标值所在地址的指针。换句话说，就是需要找到这样一个指针，这个指针指向特定目标值所在的地址，同时，这个指针还必须由程序基地址偏移得到。

**要找到基地址指针通常有2种方法：**

- 手动查找；
- 使用`Generate pointermap`和`Pointer scan for this address`进行自动查找。

## 手动查找

**操作思路：**

1. 使用精确或模糊值查找找到目标数值（本例中，即Health）所在的地址；
2. 通过`Find out what accesses this address`或者`Find out what accesses this address`查找是哪些指令在访问目标数值（本例中，即Health）的地址；
3. 通过“访问目标数值的汇编指令”推测指向目标数值（本例中，即Health）所在地址的指针的值。
4. 找到指针的值（众所周知“指针的值是一个内存地址”）后，通过“精确值扫描”查找这个地址值，找到哪个地址存放了这个地址值（通常扫描结果不止一个）。
5. 如果足够幸运，第1次扫描得到的指针中就是基地址指针，那么手动查找到底结束。但通常情况下，都必须查找多级指针（即指向指针的指针，指向指针的指针的指针...）才能找到。

### Step 2

对于Step 2，如前所述，首先使用精确值扫描找到 Health 的地址：

![](image-20240427192717805.png)

鼠标指向找到的 Health 地址，鼠标右键单击，在右键菜单中选中`Find out what accesses this address`或者`Find out what accesses this address`：

![](image-20240427193150926.png)

在当前情况下，点击 `Hit me`是修改 Health的数值，所以也算是写入操作，可以点击`Find out what writes to this address`，尝试先查找看看是哪个地址的操作在向 Health的地址`014B2E78`写入数值：

![](image-20240427193657655.png)

点击后会提示将调试器附加到当前进程。随后，CE的调试器会跟踪对Health当前地址`014B2E78`进行写入的操作码：

![](image-20240427193939860.png)

我们点击一下`Hit me`，触发一下对Health的修改：

![](image-20240427194347707.png)

鼠标双击这条指令，查看详细信息：

![](image-20240427195343453.png)

可以看到，汇编指令`sub [rbx+000007F8],eax`在修改 Health 的值。因此，我们大概可以猜到`[rbx+000007F8]`这个地址应该就是 Health 的地址，并且它是由 rbx寄存器中的值偏移`000007F8`个地址后得到的。那么，rbx 寄存器的值`014B2680`应该是一个指针的值；并且CE也提示我们，在访问当前Health地址的指针的值可能是`014B2680`。

> 在C语言中，指针作为一种特殊的数据类型，其特点是：指针的值是一个内存地址，通过使用指针，我们可以直接访问或修改指针指向的地址的值。

**既然我们已经知道了这个指针的值`014B2680`，就可以利用精确值查找的方式找到这个指针的地址。**

右键单击或者`Ctrl+C`可以方便地直接复制这个CE猜测的可能的地址值：

![](image-20240427201327543.png)

点击开启新扫描：

![](image-20240427203555631.png)

填入指针的值，开始第1次扫描：

![](image-20240427204053742.png)

> 注意：对于32位平台，指针的大小为4Bytes；64位平台指针的大小位8Bytes。

通过搜索，我们可以到有哪些地址存放了`014B2680`这个地址值。可以看到，这63个地址都存放了`014B2680`这个地址值，那么可能这63个地址都是指针。

![](image-20240427204348679.png)

有时候，搜索到的结果可能会很多，比如这一次。但幸运的是，可以看到其中有2个绿色的结果，这2个指针就是基于基地址得到的指针。我们可以将它们加入 addresslist 中测试一下是否是我们要找的：

![](image-20240427214934304.png)

双击 addresslist 中的条目，勾选`Poniter`：

![](image-20240427215129721.png)

可以看到这个基地址指针指向地址正是`014B2680`，那么可以通过地址`014B2680`偏移`000007F8`个地址获得Health所在的地址：

![](image-20240427215439053.png)

修改指针的偏移（offset），验证指针的指向：

![](image-20240427220012598.png)

可以看到，偏移后的地址正好与之前精确值搜索得到的 Health 的地址相同。取消勾选`Hexadecimal`以十进制显示地址中存放的值，发现值也与 Health 的值相同：

![](image-20240427220425180.png)

以同样的方法修改另一个基地址指针的偏移量：

![](image-20240427220831354.png)

修改后的 addresslist：

![](image-20240427220909297.png)

点击`Hit me`修改 Health的值，验证 addresslist 中的基地址指针所指向的地址中的值是否会保持一致：

![](image-20240427221052785.png)

可以看到，第一个基地址指针所指向的地址中的值的变化与Health相同，但第二个就不同了：

![](image-20240427221407502.png)

双击查看详情。可以看到，虽然基地址指针的地址没有变化，但是指针的值已经变了。

![](image-20240427222121253.png)

现在看来，基地址指针`"Tutorial-x86_64.exe"+325A70`似乎就是我们要找的。我们将Cheat Tables保存为CT文件：

![](image-20240427223149334.png)

![](image-20240427223434239.png)

关闭并重新运行 Cheat Engine Tutorial，并加载保存的CT文件：

![](image-20240427223652501.png)

可以看到，之前使用“精确值扫描”找到的地址的值现在已经没法与 CE Tutorial Step 2 的 Health 的值保持一致了，但是基地址指针指向的地址的值依然与 Health 保持一致：

![](image-20240427224024302.png)

将基地址指针指向的地址的值修改为1000，便可立即通关Step 2：

![](image-20240427224206566.png)

Step 2 的按钮`Next`现在变为可点击：

![](image-20240427224237129.png)

### Step 3

对于 Step 3, 如前所述，首先使用模糊值扫描找到 Health 的地址：

![](image-20240428221438182.png)

当然如果明确知道值减少了多少，也可以使用`decreased value by`：

![](image-20240428221758590.png)

![](image-20240428221857452.png)

![](image-20240428221936547.png)

![](image-20240428222024168.png)

![](image-20240428222146299.png)

经过验证，发现确实是这个地址保存了 Health 的值：

![](image-20240428222257932.png)

继续手动查找 Step 3 中 Health 的基地址指针。在右键菜单中选中`Find out what accesses this address`或者`Find out what accesses this address`，调用debugger，查找访问或修改了这个地址的值的操作：

![](image-20240428222703326.png)

点击`Hit me`，触发对 Health 值的修改：

![](image-20240428223127849.png)

双击查看`Extra info`：

![](image-20240428223937001.png)

使用精确值扫描查找指针：

![](image-20240428224650698.png)

虽然扫描得到的结果很多，但幸运的是，其中一个是基地址指针。我们将它加入 addresslist ，测试并验证基地址指针：

![](image-20240428225155145.png)

修改基地址指针的偏移量：

![](image-20240428230244637.png)

准备重新运行Tutorial程序，验证基地址指针：

![](image-20240428230410805.png)

重新运行Tutorial程序后，发现基地址指针所指向的地址的值在合理范围内，似乎是有效的。

![](image-20240428230713755.png)

修改基地址指针所指向地址的值，验证一下：

![](image-20240428231014800.png)

到这里，Step 3 中 Health 的基地址指针也找到了。

### Step 6

```text
Step 6: Pointers: (PW=098712)
In the previous step I explained how to use the Code finder to handle changing locations. But that method alone makes it difficult to find the address to set the values you want.
That's why there are pointers:

At the bottom you'll find 2 buttons. One will change the value, and the other changes the value AND the location of the value.
For this step you don't really need to know assembler, but it helps a lot if you do.

First find the address of the value. When you've found it use the function to find out what accesses this address.
Change the value again, and an item will show up in the list. Double click that item. (or select and click on more info) and a new window will open with detailed information on what happened when the instruction ran.
If the assembler instruction doesn't have anything between a '[' and ']' then use another item in the list.
If it does it will say what it think will be the value of the pointer you need.
Go back to the main cheat engine window (you can keep this extra info window open if you want, but if you close it, remember what is between the '[' and ']' ) and do a 4 byte scan in hexadecimal for the value the extra info told you.
When done scanning it may return 1 or a few hundred addresses. Most of the time the address you need will be the smallest one. Now click on the "Add Address Manually" button and select the pointer checkbox.

The window will change and allow you to type in the address of a pointer and an offset.
Fill in the address you just found. It can be in the form: "Tutorial-i386.exe"+xxxxxx (relative to the process), 
or you can double click the address to add it to the address list and use the absolute address which appears there.
If the assembler instruction has a calculation (e.g: [esi+12]) at the end then type the value in that's at the end above the address field. This is the offset. Otherwise leave it 0. If it was a more complicated instruction look at the following calculation.

Example of a more complicated instruction:
[EAX*2+EDX+00000310] eax=4C and edx=00801234.
In this case EDX would be the value the pointer has, and EAX*2+00000310 the offset, so the offset you'd fill in would be 2*4C+00000310=3A8. (This is all in hex, use calc.exe from Windows in Programmer mode to calculate hex values.)

Back to the tutorial, click OK and the address will be added. If all went right the address will show P->xxxxxxx, with xxxxxxx being the address of the value you found. If that's not right, you've done something wrong.
Now, change the value using the pointer you added in to 5000 and click in the 'Active' coloumn to freeze it. Then click Change pointer, and if all went right the Next button will become visible.


extra:
You could also use the pointer scanner to find the pointer to this address. https://cheatengine.org/help/pointer-scan.htm
```

Step 6 主要介绍了如何手动查找基于基地址的指针。

首先，在明确知道初始值的情况下，使用精确值扫描找到变量的直接地址：

![](image-20240503225106866.png)

![](image-20240503225239581.png)

![](image-20240503225310433.png)

由于修改变量通常是一个写入操作，我们直接查找对这个地址进行写入操作指令：

![](image-20240503225631632.png)

点击`Change value`修改值，以便 debugger 跟踪对此地址进行写入的操作指令：

![](image-20240503230106200.png)

双击条目查看详细信息：

![](image-20240503230326615.png)

可以看到，地址`015A7F80`可能是我们要找的指针的值。使用这个值进行精确值扫描，查找保存了这个值的指针：

![](image-20240503230830300.png)

找到了1个保存了这个值的指针，同时这个指针还是一个基于基地址的指针。双击将其加入 addresslist：

![](image-20240503231002732.png)

将此地址修改为指针：

![](image-20240503231651592.png)

点击`Change pointer`修改目标变量的地址，验证基于基地址的指针指向是否还依然正确：

![](image-20240503232231809.png)

修改基于基地址指针所指向的地址中的值，并勾选`Active`锁定值：

![](image-20240503232748147.png)

