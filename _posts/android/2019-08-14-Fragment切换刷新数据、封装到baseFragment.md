---
layout: post
title: Fragment切换刷新数据
categories: Android
description: ragment 刷新数据
keywords: Android, fragment、刷新
---



在BaseFragment中加入下面的代码

```

 /**
     * 正常情况下的切换   add fragment 时有效
     * @author: brian
     * @date:  2019/8/14
     **/
    override fun onHiddenChanged(hidden: Boolean) {
        // TODO Auto-generated method stub
        super.onHiddenChanged(hidden)
        if (this != null && !hidden) {
            changeToVisiable(true)
            Log.d("BaseFragment", "fragment 切换  ---")
        }
    }

    open fun changeToVisiable(isVisibleToUser: Boolean) {}

```



在子fragment中继承changeToVisiable方法即可

比如 Fragment1 继承 BaseFragment ，则在Fragment1中继承changeToVisiable 方法

```


    //切换到本fragment
    override fun changeToVisiable(isVisibleToUser: Boolean) {
        super.changeToVisiable(isVisibleToUser)
        //可刷新数据
 
    }


```

