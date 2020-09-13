---
layout: post
title: Android GPU性能分析
categories: Android
description: Android GPU 性能
keywords: Android, GPU, 渲染,性能
---
# 一、查看监控GPU渲染效果

设置-》开发者选项-》监控-》GPU呈现模式分析-》在屏幕上显示为条形图

![image-20200609150532956](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/gpu/image-20200609150532956.png)





设置-》开发者选项-》硬件加速渲染-》调试GPU过度绘制-》显示过度绘制区域



![image-20200609160350122](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/gpu/image-20200609160350122.png)



![image-20200609192103834](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/gpu/image-20200609192103834.png)



# 二、通常的优化手段

提高GPU的渲染主要从以下几点入手：
1、移除布局中不需要的背景。

2、使视图层次结构扁平化。

3、降低透明度。



# 三、分析UI

经分析发现，多了两个不需要的层级，如下图，去掉两个顶层层级即可提高GPU渲染性能

![image-20200609160208003](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/gpu/image-20200609160208003.png)



