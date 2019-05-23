---
layout: post
title: Android studio 的热键被占用
categories: SmallProblem
description: ctrl+shift+f 热键被输入法占用
keywords: ctrl+shift+f, 热键，Android Studio

---

最近在Android Studio的使用过程中，发现热键” ctrl + shift + f “偶尔失效，查找到两种方案来解决这个问题：

1、使用Windows Hotkey Explorer 来查看所有的热键情况。这个方法对window10支持不好，找不出占用 ctrl + shift + f 程序，故解决不了我的问题。

2、使用PChunter 查看每个进程的热键，这个需要手动查看每个进程的情况，比较耗时，依然找不到 ctrl + shift + f 的占用情况。

最后，偶然发现的切换输入法时这个热键会生效，后来确定了搜狗输入法占用了这个热键，前往搜狗输入法的 工具箱-属性设置-按键-系统功能快捷键 取消对于的快捷键即可。

题外话：
	至于为什么耗费这么多时间来处理这个按键的问题，主要原因是开发过程中全局搜索这个功能经常使，快捷键呼出只需1s时间，而通过导航栏来点击则需要6s的时间，若1天使用该功能20次，每年工作日约250天，则1年所浪费的时间为 20x5x250/60x60 ≈ 7小时



