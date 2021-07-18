---
title: leetcode-2 两数相加
date: 2021-06-19 18:24:59
tags: leetcode
categories: algoririthms
---
## 题目描述
给你两个**非空**的链表，表示两个非负的整数。它们每位数字都是按照**逆序**的方式存储的，并且每个节点只能存储**一位**数字。
<!-- more -->
请你将两个数相加，并以相同形式返回一个表示和的链表。你可以假设除了数字 0 之外，这两个数都不会以 0 开头。
示例 1：
![addtwonumber](/img/addtwonumber1.jpg)
```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```
示例 2：
```
输入：l1 = [0], l2 = [0]
输出：[0]
```
示例 3：
```
输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
```

**提示**：

每个链表中的节点数在范围 [1, 100] 内 0 <= Node.val <= 9，题目数据保证列表表示的数字不含前导零。

## 求解
本题的思路还是比较简单的，只需要对链表的结构足够了解，基本能够求解，如下代码：
``` golang
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    retListNode := &ListNode{}
    next := retListNode
    pre := &ListNode{}
    carry := 0 // 进位值
    // 首先遍历指针，将值相加，注意进位
    for l1 != nil && l2 != nil {
        pre = next
        val := l1.Val + l2.Val + carry
        if val >= 10 {
            carry = 1
            val -= 10
        } else {
            carry = 0
        }
        pre.Val = val
        next = &ListNode{}
        pre.Next = next
        l1 = l1.Next
        l2 = l2.Next
    }
    // 说明l2结束了，但是l1还剩元素
    for l1 != nil {
        pre = next
        val := l1.Val + carry
        if val >= 10 {
            carry = 1
            val -= 10
        } else {
            carry = 0
        }
        pre.Val = val
        next = &ListNode{}
        pre.Next = next
        l1 = l1.Next
    }
    // 说明l1结束了，但是l2还剩元素
    for l2 != nil {
        pre = next
        val := l2.Val + carry
        if val >= 10 {
            carry = 1
            val -= 10
        } else {
            carry = 0
        }
        pre.Val = val
        next = &ListNode{}
        pre.Next = next
        l2 = l2.Next
    }
    // 最后判断是否有进位，如果有则需将进位填写到最后的数值中
    // 如果没有，需要将最后一个节点置空
    if carry == 1 {
        pre.Next.Val = 1
    } else {
        pre.Next = nil
    }
    return retListNode
}
```
需要注意的是，最后一步若`carry != 1`，不能用`next = nil`，这考察到了对指针的理解：即将`next`改变（指针本身），并未将`pre.Next`改变。