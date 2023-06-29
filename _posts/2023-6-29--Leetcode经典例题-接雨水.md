---
layout:     post
title:      Leetcode经典例题--接雨水
subtitle:   
date:       2023-6-29
author:    xuoneyuan
header-img: 
catalog: false
tags:
    - 算法
---

## 题目
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

## 思路
这个题目是一个经典的雨水积累问题，算法采用双指针来解决

1. 遍历给定的柱子高度图，记录每个位置上的左边最高柱子和右边最高柱子的高度。
2. 创建一个与给定柱子高度图相同大小的辅助数组，用于存储每个位置上的积水量。
3. 从左到右遍历柱子高度图，对于每个位置，计算该位置上的积水量。积水量等于左边最高柱子高度和右边最高柱子高度中较小的那个减去当前位置上柱子的高度。如果该积水量大于零，则将其累加到辅助数组的相应位置。
4. 最后，遍历辅助数组，将所有位置上的积水量相加，得到总的积水量。

## code
~~~
func trap(height []int) int {
if len(height) < 3 {
        return 0
    }

    left := 0
    right := len(height) - 1
    leftMax := 0
    rightMax := 0
    water := 0

    for left < right {
        if height[left] < height[right] {
            if height[left] > leftMax {
                leftMax = height[left]
            } else {
                water += leftMax - height[left]
            }
            left++
        } else {
            if height[right] > rightMax {
                rightMax = height[right]
            } else {
                water += rightMax - height[right]
            }
            right--
        }
    }

    return water
}
~~~
