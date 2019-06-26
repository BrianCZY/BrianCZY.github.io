---
layout: post
title: Linearlayout设置selector无效问题
categories: Android
description: 
keywords: Android, selector，无效，LinearLayout
---




1、selector 文件为:

selector_item_bg.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="@color/transparent_00" />
            <size android:height="30dp" />
            <corners android:radius="0dp" />
        </shape>
    </item>
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <solid android:color="@color/transparent_60" />
            <size android:height="30dp" />
            <corner android:radius="0dp" />
        </shape>
    </item>
</selector>

```

现象是在控件上使用可正常显示效果，但是在ViewGroup上使用则无效

调用方式为：

```xml
<LinearLayout
                android:id="@+id/ll_test"
                android:clickable="true"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="@drawable/selector_item_bg"
                android:orientation="horizontal">

            </LinearLayout>

```

经测试，是由于selector 的第一个 item 没有设置state_pressed，修改的方式是添加 

<item android:state_pressed="false">

修改后的selector_item_bg.xml  如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="false">
        <shape android:shape="rectangle">
            <solid android:color="@color/transparent_00" />
            <size android:height="30dp" />
            <corners android:radius="0dp" />
        </shape>
    </item>
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <solid android:color="@color/transparent_60" />
            <size android:height="30dp" />
            <corner android:radius="0dp" />
        </shape>
    </item>
</selector>


```

