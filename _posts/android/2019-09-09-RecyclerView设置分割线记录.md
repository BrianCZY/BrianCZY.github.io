---
layout: post
title: RecyclerView设置分割线记录
categories: Android
description: RecyclerView 分割线设置，有左右边距
keywords: Android, RecyclerView, DividerItemDecoration
---



java 代码上设置 drawable

```java

  //添加自定义分割线
 DividerItemDecoration divider = new DividerItemDecoration(this,DividerItemDecoration.VERTICAL);
 divider.setDrawable(ContextCompat.getDrawable(this,R.drawable.custom_divider));
 recycler.addItemDecoration(divider);

```



custom_divider.xml 的内容如下：

```xml

<?xml version="1.0" encoding="utf-8"?>

<inset xmlns:android="http://schemas.android.com/apk/res/android"
    android:insetLeft="@dimen/dp_16"
    android:insetRight="@dimen/dp_16">
    <!--insetLeft 即为左边距的数值-->
    <shape android:shape="rectangle">
        <solid android:color="@color/gray_e5e5e5" />
        <size android:height="1dp" />

    </shape>
</inset>

```

