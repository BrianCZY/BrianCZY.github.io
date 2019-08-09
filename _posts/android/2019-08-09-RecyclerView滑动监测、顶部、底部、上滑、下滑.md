---
layout: post
title: RecyclerView滑动监测、顶部、底部、上滑、下滑
categories: Android
description: 监测RecyclerView的滑动，并判断是否到达顶部、底部、上滑、下滑
keywords: Android, RecyclerView、RecyclerView滑动监测
---



```kotlin


        recycler_view.setOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                if (!recyclerView.canScrollVertically(-1)) {
                    Log.d(TAG, "recyclerView  顶部--")
                } else if (!recyclerView.canScrollVertically(1)) {
                    Log.d(TAG, "recyclerView  底部--")
                } else if (dy < 0) {
                    Log.d(TAG, "上划")
                } else if (dy > 0) {
                    Log.d(TAG, "下划")
                }
            }
        })

```



canScrollVertically(int direction) 函数的解释如下： 大意为判断是否可以滑动指定的像素， direction  为负向上，direction 为正向下

```


    /**
     * Check if this view can be scrolled vertically in a certain direction.
     *
     * @param direction Negative to check scrolling up, positive to check scrolling down.
     * @return true if this view can be scrolled in the specified direction, false otherwise.
     */
    public boolean canScrollVertically(int direction) {
        final int offset = computeVerticalScrollOffset();
        final int range = computeVerticalScrollRange() - computeVerticalScrollExtent();
        if (range == 0) return false;
        if (direction < 0) {
            return offset > 0;
        } else {
            return offset < range - 1;
        }
    }


```

