---
layout: post
title: RecyclerView自适应大小
categories: Android
description: 根据item数量调整RecyclerView的高度
keywords: Android, RecyclerView

---

需求描述：1、RecylerView嵌套于ScrollView；

​					2、RecyclerView初始化时只显示几条内容或不显示内容；

​					3、每次点击一个Button，就往RecyclerView追加内容，RecyclerView高度也会增大



具体方案：

步骤1：设置isAutoMeasureEnabled = true      

  isAutoMeasureEnabled 的解释为:   Defines whether the measuring pass of layout should use the AutoMeasure mechanism of* {@link RecyclerView} or if it should be done by the LayoutManager's implementation of* {@link LayoutManager#onMeasure(Recycler, State, int, int)}  。也就是设置为true的时候会自动Measure 宽高。

```kotlin

    val linearLayoutManager = LinearLayoutManager(context)
    linearLayoutManager.isAutoMeasureEnabled = true
    recycler_view.layoutManager = linearLayoutManager
    //禁止recyclerView 的滑动事件 避免跟ScrollView冲突
    // recycler_view.setNestedScrollingEnabled(false)  

```

步骤2：RecyclerView 外部套一层RelativeLayout  ，高度设置为 android:layout_height="wrap_content"

比如我的整个布局结构为

```xml
<ScrollView>
	 <LinearLayout>
	   <RelativeLayout>
            <RecyclerView/>
	     </RelativeLayout>
	 </LinearLayout>
</ScrollView>
```

具体代码如下：


```xml
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recycler_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:background="@color/white">

            </androidx.recyclerview.widget.RecyclerView>
        </RelativeLayout>
```



