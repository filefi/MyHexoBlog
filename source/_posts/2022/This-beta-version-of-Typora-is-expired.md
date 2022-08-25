---
title: This beta version of Typora is expired
date: 2022-08-25 21:33:53
updated: 2022-08-25 21:33:53
tags: [Editor]
categories: Miscellaneous
---

<!-- more -->

![](dealwith0.webp)

通过修改注册表，可以解决这个问题：

1. win+r 打开运行窗口
2. 在搜索栏输入`regedit`，回车后打开注册表
3. 在注册表中找到：`计算机\HKEY_CURRENT_USER\Software\Typora`
4. 鼠标右键Typora，选择权限
5. 选择`Administraors`，将下面的权限选择为拒绝

![](dealwith.webp)

6. 重新打开Typora，发现可以正常运行了
