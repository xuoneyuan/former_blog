---
layout:     post
title:      test
subtitle:   
date:       2023-3-25
author:    xuoneyuan
header-img: 
catalog: false
tags:
    - 算法
---

### 题目
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

### 思路
1. 首先判断给定的柱子高度图长度是否小于3，如果小于3，则无法形成积水，直接返回0
2. 初始化左指针 left 和右指针 right 分别指向柱子高度图的最左端和最右端
3. 初始化左边最高柱子高度 leftMax 和右边最高柱子高度 rightMax，初始值都为0
4. 初始化变量 water 用于记录积水量，初始值为0
5. 使用双指针遍历柱子高度图，当 left 小于 right 时进行循环
6. 在每次循环中，比较 height[left] 和 height[right] 的高度，选择较小的一侧开始计算
7. 如果 height[left] 小于 height[right]，则表示左侧的柱子可以作为积水的左边界。接着判断 height[left] 是否大于 leftMax，如果大于，则更新 leftMax 的值为 height[left]；如果小于或等于 leftMax，则说明可以积水，将积水量累加到 water 中。然后将左指针 left 向右移动一格
8. 如果 height[left] 大于等于 height[right]，则表示右侧的柱子可以作为积水的右边界。接着判断 height[right] 是否大于 rightMax，如果大于，则更新 rightMax 的值为 height[right]；如果小于或等于 rightMax，则说明可以积水，将积水量累加到 water 中。然后将右指针 right 向左移动一格
9. 循环结束后，返回累加的积水量 water

### 题解
#### go
~~~go
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
#### cpp
~~~cpp
class Solution {
  public:
    int trap(vector<int>& height) {
        int n = height.size();
        if (n == 0) return 0;

        int left = 0, right = n - 1;
        int leftMax = height[left], rightMax = height[right];
        int water = 0;

        while (left < right) {
            if (height[left] < height[right]) {
                if (height[left] >= leftMax) {
                    leftMax = height[left];
                } else {
                    water += leftMax - height[left];
                }
                left++;
            } else {
                if (height[right] >= rightMax) {
                    rightMax = height[right];
                } else {
                    water += rightMax - height[right];
                }
                right--;
            }
        }

        return water;
    }
};
~~~
