---
layout: post
title:  "选择排序"
date:   2019-04-28
excerpt: "基于冒泡排序的简单排序算法"
tag:
- algorithm
- sort
- java
comments: true
---

# 介绍
冒泡排序每次比较大小时可能会交换位置,最终目的是将本轮循环中最大的值移到末尾。
如果交换的消耗很大的话,会影响程序性能(java交换只是改变引用),所以选择排序思想是用一个变量记录循环中最大的值(的位置),等循环结束将最大值排到末尾就可以了。

# 思路

1. 冒泡排序算法为
   ```java
    void bubbleSort(int[] datas) {
       int end = datas.length - 2;
        for (int i = end; i >= 1; i--) {
            for (int j = 0; j <= i; j++) {
                //如果data[j] > data[j+1] 交换两个数字
                if (datas[j] > datas[j + 1]) {
                    int temp = datas[j];
                    datas[j] = datas[j + 1];
                    datas[j + 1] = temp;
                }
            }
        }
   }
   
   ```
   ---
2. 修改内层循环不涉及交换,只记录最大值(默认最大值为外层循环变量i)
   ```java
    int maxIndex = i;
    for (int j = 0; j <= i - 1; j++) {
        //跟目前最大的maxIndex比较
        if (data[j] > data[maxIndex]) {
            maxIndex = j;
        }
    }
   ```
   ---
3. 结束之后交换i和maxIndex对应数组值
   ```java
   public static void swap(int[] data, int x, int y) {
       int temp = data[x];
       data[x] = data[y];
       data[y] = temp;
   }
   ```
   ---
4. 最终代码如下
   ```java
   public static void selectSort(int[] data) {
        int end = data.length - 1;
        for (int i = end; i >= 1; i--) {
            int maxIndex = i;
            for (int j = 0; j <= i - 1; j++) {
                if (data[j] > data[maxIndex]) {
                    maxIndex = j;
                }
            }
            swap(data, i, maxIndex);
        }
    }
   ```
