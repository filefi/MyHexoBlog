---
title: Cheat Engine Tutorial 学习笔记1：精确/模糊值扫描，手动查找指针、多级指针与基地址指针
date: 2022-02-16 21:41:21
tags: [PWN, Reverse, CheatEngine]
categories: Reverse
---



本文重点介绍：

- 精确/模糊数值扫描
- 手动查找指针、多级指针与基地址指针

CE版本如下：

- Cheat Engine Version: 7.4和7.5

- Tutorial Version: v3.6



<!-- more -->

# 前言

## Step 1: 

![](image-20220215145201605.png)

### Description
```text
Welcome to the Cheat Engine Tutorial (v3.6)

This tutorial will teach you the basics of cheating in video games. It will also show you foundational aspects of using Cheat Engine (or CE for short). Follow the steps below to get started.

1: Open Cheat Engine if it currently isn't running.
2: Click on the "Open Process" icon (it's the top-left icon with the computer on it, below "File".).
3: With the Process List window now open, look for this tutorial's process in the list. It will look something like "00001F98-Tutorial-x86_64.exe" or "0000047C-Tutorial-i386.exe". (The first 8 numbers/letters will probably be different.)
4: Once you've found the process, click on it to select it, then click the "Open" button. (Don't worry about all the other buttons right now. You can learn about them later if you're interested.)

Congratulations! If you did everything correctly, the process window should be gone with Cheat Engine now attached to the tutorial (you will see the process name towards the top-center of CE).

Click the "Next" button below to continue, or fill in the password and click the "OK" button to proceed to that step.)

If you're having problems, simply head over to forum.cheatengine.org, then click on "Tutorials" to view beginner-friendly guides!
```

### 操作

步骤1：选择要附加的进程

![](image-20220215145836858.png)

![](image-20220215150314077.png)

<!-- more -->

# 精确值扫描

## Step 2

```
Step 2: Exact Value scanning (PW=090453)
Now that you have opened the tutorial with Cheat Engine let's get on with the next step.

You can see at the bottom of this window is the text Health: xxx
Each time you click 'Hit me'  your health gets decreased.

To get to the next step you have to find this value and change it to 1000

To find the value there are different ways, but I'll tell you about the easiest, 'Exact Value':
First make sure value type is set to at least 2-bytes or 4-bytes. 1-byte will also work, but you'll run into an easy to fix problem when you've found the address and want to change it. The 8-byte may perhaps works if the bytes after the address are 0, but I wouldn't take the bet.
Single, double, and the other scans just don't work, because they store the value in a different way.

When the value type is set correctly, make sure the scantype is set to 'Exact Value'
Then fill in the number your health is in the value box. And click 'First Scan'
After a while (if you have an extremely slow pc) the scan is done and the results are shown in the list on the left.

If you find more than 1 address and you don't know for sure which address it is, click 'Hit me', fill in the new health value into the value box, and click 'Next Scan'
repeat this until you're sure you've found it. (that includes that there's only 1 address in the list.....)

Now double click the address in the list on the left. This makes the address pop-up in the list at the bottom, showing you the current value.
Double click the value, (or select it and press enter), and change the value to 1000.

If everything went ok the Next button should become enabled, and you're ready for the next step.


Note:
If you did anything wrong while scanning, click "New Scan" and repeat the scanning again.
Also, try playing around with the value and click 'Hit me'.
```

**操作：**

![](image-20220215151801088.png)

![](image-20220215152155476.png)

![](image-20220215152500187.png)

![](image-20220215152558471.png)

![](image-20220215152638141.png)

![](image-20220215152855906.png)

![](image-20220215153108418.png)

![](image-20220215153634400.png)

![](image-20220215153905170.png)

![](image-20220215154113896.png)

![](image-20220215154508980.png)

# 模糊值扫描

## Step 3

```
Step 3: Unknown initial value (PW=419482)
Ok, seeing that you've figured out how to find a value using exact value let's move on to the next step.

First things first though. Since you are doing a new scan, you have to click on New Scan first, to start a new scan. (You may think this is straighforward, but you'd be surprised how many people get stuck on that step) I won't be explaining this step again, so keep this in mind.
Now that you've started a new scan, let's continue.

In the previous test we knew the initial value so we could do an exact value search, but now we have a status bar where we don't know the starting value.
We only know that the value is between 0 and 500. And each time you click 'Hit me' you lose some health. The amount you lose each time is shown above the status bar.

Again there are several different ways to find the value. (like doing a decreased value by... scan), but I'll only explain the easiest. "Unknown initial value", and decreased value.
Because you don't know the value it is right now, exact value wont do any good, so choose as scantype 'Unknown initial value', again, the value type is 4-bytes. (Most windows apps use 4-bytes.) Click First scan and wait till it's done.

When it is done click 'Hit me'. You'll lose some of your health. (the amount you lost shows for a few seconds and then disappears, but you don't need that)
Now go to Cheat Engine, and choose 'Decreased Value' and click 'Next Scan'
When that scan is done, click 'Hit me' again, and repeat the above till you only find a few. 

We know the value is between 0 and 500, so pick the one that is most likely the address we need, and add it to the list.
Now change the health to 5000, to proceed to the next step.
```

**操作：**

![](image-20220215173831872.png)

![](image-20220215173929270.png)![](image-20220215174056856.png)

![](image-20220215174649344.png)

![](image-20220215174800299.png)![](image-20220215174937499.png)![](image-20220215175020697.png)



# 指针、多级指针与基地址指针

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
