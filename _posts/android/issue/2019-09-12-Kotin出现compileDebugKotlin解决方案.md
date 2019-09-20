---
layout: post
title: Kotin出现app:compileDebugKotlin解决方案
categories: Android
description: 
keywords: compileDebugKotlin

---

问题：Caused by: org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':app:compileDebugKotlin'.

解决方案：

步骤1、点击 assemble，重新run

![1568280067054](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/compile_debug_kotlin/1568280067054.png)



步骤2：会出现如下的错误信息，此时问题已经找到了

![1568280045090](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/compile_debug_kotlin/1568280045090.png)