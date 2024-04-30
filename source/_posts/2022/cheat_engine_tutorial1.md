---
title: Cheat Engine Tutorial 学习笔记1：精确/模糊值扫描
date: 2022-02-16 21:41:21
tags: [PWN, Reverse, CheatEngine]
categories: Reverse
---



本文重点介绍：

- 精确/模糊数值扫描

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

## Step 4

```text
Step 4: Floating points (PW=890124)
In the previous tutorial we used bytes to scan, but some games store information in so called 'floating point' notations. 
(probably to prevent simple memory scanners from finding it the easy way)
a floating point is a value with some digits behind the point. (like 5.12 or 11321.1)

Below you see your health and ammo. Both are stored as Floating point notations, but health is stored as a float and ammo is stored as a double.
Click on hit me to lose some health, and on shoot to decrease your ammo with 0.5
 
You have to set BOTH values to 5000 or higher to proceed.

Exact value scan will work fine here, but you may want to experiment with other types too.



Hint: It is recommended to disable "Fast Scan" for type double
```

Step 4 主要是介绍了 float 和 double 值类型的精确值扫描，基本操作和整型的精确值扫描类似。

![](image-20240430100922143.png)

执行`Exact Value`扫描，并将`Value Type`设为`Float`，填入要扫描的值，开始扫描：

![](image-20240430101837596.png)

点击`Hit me`修改 Health 的值：

![](image-20240430102319944.png)

填入修改后的值，执行下一次精确值扫描：

![](image-20240430102652795.png)

将得到的备选地址加入 addresslist 进行验证：

![](image-20240430103435764.png)

修改地址保存的值之后，需要点击`Hit me`触发一下显示的值的修改：

![](image-20240430103942060.png)

可以看到，点击`Hit me`之后，地址中保存的值也相应地变化了：

![](image-20240430104225544.png)

开始新扫描，准备扫描 Ammo 的地址：

![](image-20240430104504605.png)

设置值类型为`Double`，填入值，开始首次扫描：

![](image-20240430104955721.png)

点击`Fire`修改 Ammo 的值：

![](image-20240430105152459.png)

填入修改后的值，进行下一次扫描：

![](image-20240430105312568.png)

得到两个备选地址：

![](image-20240430105529833.png)

再次修改值，以便对备选地址进行过滤：

![](image-20240430105631230.png)

填入修改后的值，进行下一次扫描：

![](image-20240430105741906.png)

依然还是得到两个地址，将它们加入 addresslist 逐个进行验证：

![](image-20240430105850638.png)

修改地址中的值：

![](image-20240430110323596.png)

可以看到，点击`Fire`后，addresslist中的第一个备选地址中的值也相应地改变了，而第二个备选地址中的值则没有变化。那么，基本可以断定，第一个备选地址是我们要找的。

![](image-20240430110755464.png)

修改第二个备选地址中的值，对第二个备选地址进行验证：

![](image-20240430111148365.png)

修改第二个备选地址中的值后，点击`Fire`触发 Ammo 显示值的变化。可以看到，修改第二个备选地址中的值后，没有对 Ammo 的值产生影响，点击`Fire`后，第二个备选地址中值依旧没有变化。那么，现在可以确定第一个备选地址是我们要找的，第二个则不是。

![](image-20240430111509913.png)

将第二个备选地址从 addresslist 中删除：

![](image-20240430111946289.png)

接下来，由于 Step 4 这一关需要 Health 和 Ammo两个值同时大于或等于5000，我们再修改一下 Health 地址中的值，使其满足过关条件：

![](image-20240430112302524.png)

修改 Health 地址中的值后，甚至还没有点击`Hit me`触发显示的值变化，Next 按钮就已经可以点击了：

![](image-20240430112531318.png)

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

