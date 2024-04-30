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

