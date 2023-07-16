---
layout:     post
title:      Leetcode经典例题——合并K个升序链表
subtitle:   
date:       2023-7-16
author:     xuoneyuan
header-img: 
catalog: false
tags:
    - 算法
---

### 题目
给你一个链表数组，每个链表都已经按升序排列。\
请你将所有链表合并到一个升序链表中，返回合并后的链表。

示例 ：\
输入：lists = [[1,4,5],[1,3,4],[2,6]]\
输出：[1,1,2,3,4,4,5,6]\
解释：链表数组如下：\
[\
  1->4->5,\
  1->3->4,\
  2->6\
]\
将它们合并到一个有序链表中得到\
1->1->2->3->4->4->5->6

### 分析
#### 分治法
该方法的基本思路如下：\
将链表数组分成两半，递归地将左半部分和右半部分进行合并，得到两个合并后的有序链表。\
将两个合并后的有序链表再进行合并，得到最终的合并后的有序链表。\
这个过程可以不断递归地进行，直到链表数组只剩下一个链表或为空。\
以下是使用分治法解决问题的详细步骤：\
检查边界情况：如果链表数组为空，直接返回 nil。\
调用递归函数 mergeLists，将链表数组和起始索引、结束索引作为参数传递给它。

在 mergeLists 函数中：\
如果起始索引等于结束索引，表示只剩下一个链表，直接返回该链表。\
计算中间索引 mid，将链表数组分成两半。\
递归调用 mergeLists 函数，分别传入左半部分链表数组和右半部分链表数组，得到两个合并后的有序链表。\
调用 mergeTwoLists 函数，将两个合并后的有序链表进行合并，返回合并后的结果。

在 mergeTwoLists 函数中：\
创建一个 dummy 节点作为合并后链表的头节点，并使用一个指针 curr 指向当前节点。\
使用两个指针 l1 和 l2 分别指向两个链表的当前节点。\
循环比较 l1 和 l2 的节点值，将较小值的节点添加到合并后的链表中，并将相应指针向后移动。\
最后，将剩余的节点直接连接到合并后的链表末尾。\
返回 dummy.Next 即可得到合并后的链表。

### code
~~~
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeKLists(lists []*ListNode) *ListNode {

    if len(lists) == 0 {
        return nil
    }

    interval := 1
    for interval < len(lists) {
        for i := 0; i < len(lists)-interval; i += interval * 2 {
            lists[i] = mergeTwoLists(lists[i], lists[i+interval])
        }
        interval *= 2
    }

    return lists[0]
}

// 合并两个有序链表
func mergeTwoLists(l1, l2 *ListNode) *ListNode {
    dummy := &ListNode{}
    curr := dummy

    for l1 != nil && l2 != nil {
        if l1.Val < l2.Val {
            curr.Next = l1
            l1 = l1.Next
        } else {
            curr.Next = l2
            l2 = l2.Next
        }
        curr = curr.Next
    }

    if l1 != nil {
        curr.Next = l1
    } else {
        curr.Next = l2
    }

    return dummy.Next
}
~~~






