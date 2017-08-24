---
title: 2.Add Two Numbers
date: 2017-08-13 18:34:24
tags: [LeetCode]
category: LeetCode刷题
---

## Description:


> You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

> You may assume the two numbers do not contain any leading zero, except the number 0 itself.

> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
> Output: 7 -> 0 -> 8


LeetCode link: [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/description/)

## Solution:

<!-- more -->

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l = new ListNode(0);
        ListNode start = l;
        int nVal = 0;
        while(l1 != null || l2 != null) {
            int l1Val = l1 == null ? 0 : l1.val;
            int l2Val = l2 == null ? 0 : l2.val;
            
            int val = (l1Val + l2Val + nVal) % 10;
            nVal = (l1Val + l2Val + nVal) / 10;
            
            start.next = new ListNode(val);
            start = start.next;
            
            if (l1 != null) {
                l1 = l1.next;
            }
            if (l2 != null) {
                l2 = l2.next;
            }
            
        }
        if (nVal > 0) {
            start.next = new ListNode(nVal);
        }
        return l.next;
    }
}
```