---
layout: post
title: ViewPager切换Fragment数据刷新问题解决笔记
categories: Android
description: 
keywords: Viewpager刷新数据, Viewpager Fragment切换刷新数据

---

需求描述：一共有3个Fragment，切换时只需要刷新其中一个Fragment或多个Fragment的数据（不需要全部刷新）

解决方案：

在需要刷新的数据的Fragment添加如下代码：

```kotlin
  var isCreate = false
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        isCreate = true
    }

    /**
     * viewpager 切换本Fragment时，会调用这个函数，即是否可见，另外需要注意setUserVisibleHint 一开始的状态不准确，
     * 需要配合onViewCreated 来判断何时来刷新数据
     */
    override fun setUserVisibleHint(isVisibleToUser: Boolean) {
        super.setUserVisibleHint(isVisibleToUser)
        Log.d(TAG, "setUserVisibleHint isVisibleToUser:$isVisibleToUser")
        if (isVisibleToUser && isCreate) {
            //requestOwnDevice()
            //这里执行  刷新数据（ 网络请求、加载数据库数据、更改UI等）
        }
    }
```

