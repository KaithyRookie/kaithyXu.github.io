---
layout:     post
title:      LeetCode环形链表II
subtitle:   环形链表II
date:       2019-09-17
author:     KaithyXu
header-img: img/LinkedListCycle.jpg
catalog: true
tags:
    - algorithm
---
## 环形链表II


### 题目描述

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

说明：不允许修改给定的链表。

示例 1：

输入：head = [3,2,0,-4], pos = 1
输出：tail connects to node index 1
解释：链表中有一个环，其尾部连接到第二个节点。

### 解题思路

首先利用快慢指针判断链表是否有环，如果在移动中，快指针与慢指针相等，则说明链表存在一个环，否则链表没有环，移动快指针时需要注意当前的元素是否为null，不然会抛出异常。
若在移动中快慢指针相遇，则此时快指针移动的长度一定是慢指针的移动的路径长度的两倍，设链表入环的第一个节点的位置距头节点长度为a，两指针相遇时的节点距入环的第一个节点的距离为x，
慢指针走的路径长度：a+x
快指针走的路径长度：a+x+mb
2(a+x) = a+x+mb
=> a+x = mb
此时如果从头节点开始移动a个节点会到达入环的第一个节点，而从两指针相遇的位置走a个节点,此时快指针已经走过了2a+x+mb长度，
2a+x+mb => a+(a+x)+mb => a+2mb 此时相当于快节点已经回到到链表入环节点，由于此时快指针一次只走一步，所以可让一个临时指针从头节点开始移动，每次每个指针只移动一步，当临时节点与快节点相遇时，就是所要找的入口节点

### 代码实现

```
/**
 *
 * 环形链表II
 * @author kaithy.xu
 * @date 2019-09-17 18:24
 */
public class LinkedListCycleII {

    public ListNode detectCycle(ListNode head) {

        ListNode fast = head;
        ListNode slow = head;
        while (fast != null) {
            fast = fast.next;
            slow = slow.next;
            if(fast == null) {
                return null;
            }
            fast = fast.next;
            if(fast == slow) {
                break;
            }
        }
        if(fast == null) {
            return null;
        }
        ListNode record = head;
        while (record != slow) {
            slow = slow.next;
            record = record.next;
        }
        return record;
    }
}


```

