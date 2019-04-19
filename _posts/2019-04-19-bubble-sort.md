---
layout: post
title:  "冒泡排序"
date:   2019-04-19
excerpt: "简单的排序算法"
tag:
- algorithm
- sort
- java
comments: true
---

# 介绍
冒泡排序是一种简单的交换排序算法,思路是每次排列将最大的一项排到序列最后,下次循环找出除了最后一项之外的最大项,排到倒数第二的位置,依次下去，直至最后将第二小的数据排到第二的位置,至此序列完全有序.

# 思路

1. 第一轮循环将最大的数字排到序列最后
   ```java
    void onceSort(int[] datas) {
       int end = datas.length - 2;
        for (int i = 0; i <= end; i++) {
            //如果data[i] > data[i+1] 交换两个数字
            if (datas[i] > datas[i + 1]) {
                int temp = datas[i];
                datas[i] = datas[i + 1];
                datas[i + 1] = temp;
            }
        }
   }
   ```
2. 第二次循环改变上面函数的end,将其减1,直至1,以此类推,外层循环则为下面这格式
   ```java
   for (int i = end; i >=- 1; i--) {
    // ...
   }
   ```
3. 合并内外层循环合并,排序算法为
   ```java
    void bubbleSort(int[] datas) {
       int end = datas.length - 2;
        for (int i = end; i >= 1; i--) {
            for (int j = 0; j <= i; j++) {
                //如果data[i] > data[i+1] 交换两个数字
                if (datas[j] > datas[j + 1]) {
                    int temp = datas[j];
                    datas[j] = datas[j + 1];
                    datas[j + 1] = temp;
                }
            }
        }
   }
   
   ```