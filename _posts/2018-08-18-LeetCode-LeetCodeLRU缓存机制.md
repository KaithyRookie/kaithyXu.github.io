---
layout:     post
title:      LeetCodeLRU缓存机制
subtitle:   LRU缓存机制
date:       2018-08-18
author:     KaithyXu
header-img: img/binaryTreeAncestor.jpg
catalog: true
tags:
    - algorithm
---
## LRU缓存机制


### 题目描述

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

### 解题思路

使用链表结构实现LRU缓存机制，每一次get操作，都遍历链表找到对应的节点，然后就其移动至链表的尾部，push操作时，判断容量是否已达到最大值，如果是，则从链表的头部删除一个节点，然后再将新的节点插入到链表尾部

### 代码实现

```
public class LRUAlgorithmthim {

    private int capacity; //容量

    private int cursor; //已存放的节点数量

    private ListNode node; //链表的尾部

    //静态内部类实现一个双向链表
    private static class ListNode{ 
        private int key;
        private int value;
        private ListNode next;
        private ListNode previous;

        ListNode (int key,int value){
            this.key = key;
            this.value = value;
        }
    }

    public LRUAlgorithmthim(int capacity) {
        this.capacity = capacity;
        cursor = 0;
        this.node = new ListNode(-1,-1);
    }

    public int get(int key) {
        return find(key);
    }

    //放入yield节点时，如果是新的key且容量已达到最大值，则遍历至链表头部将对应的value覆盖为-1
    public void put(int key, int value) {
        ListNode insert = new ListNode(key,value);
        if(find(key) == -1){
            node.next = insert;
            insert.previous = node;
            node = node.next;
            if(cursor == capacity){
                ListNode temp = node;
                while (temp.previous!= null && temp.previous.value != -1){
                    temp = temp.previous;
                }
                temp.value = -1;
                cursor--;
            }

            cursor++;
        }else {
            this.node.value = value;
        }
    }

    //一共分为两步，找到所需要的那个节点，然后将其移动至链表尾部
    private int find(int key){
        ListNode temp  = node;
        while (temp != null && temp.value != -1){

            if(temp.key == key){

                if(temp.next == null){
                    return temp.value;
                }
                ListNode previous = temp.previous;
                previous.next = null;
                temp.previous = null;

                ListNode nextNode = temp.next;
                nextNode.previous = null;
                temp.next = null;

                previous.next = nextNode;
                nextNode.previous = previous;
                while (nextNode.next != null){
                    nextNode = nextNode.next;
                }
                nextNode.next= temp;
                temp.previous = nextNode;
                this.node = nextNode.next;

                return  this.node.value;
            }else {
                temp = temp.previous;
            }
        }
        return -1;
    }

}


```

### 代码优化

在上面的基础上，由于find()方法需要遍历整个链表寻找元素，时间复杂度为O(n)，所以考虑使用HashMap进行键值对的存储，其次，研究了别人的优秀答案后发现自己对于哨兵节点的使用存在误区，按理说哨兵节点是不需要移动的，也就是所有节点的新增与删除都在尾部的哨兵节点之前，头部的哨兵节点之后进行，由于链表的插入时间复杂度是O(1)，所以我加了两个哨兵节点，header与 footer,同时将节点的新增与删除方法提取出来，病将find()中的移动节点至尾部的功能也提取了出来。

```
import java.util.HashMap;
import java.util.Map;

public class LRUCache {

    private int capacity;

    private int cursor;

    private ListNode header;

    private ListNode footer;

    private static class ListNode{
        private int key;
        private int value;
        private ListNode next;
        private ListNode previous;

        ListNode (int key,int value){
            this.key = key;
            this.value = value;
        }
    }

    private Map<Integer,ListNode> cache = new HashMap<>();

    public LRUCache(int capacity) {
        header = new ListNode(-1,-1);
        footer = new ListNode(-1,-1);
        header.next = footer;
        footer.previous = header;
        this.capacity = capacity;
        cursor = 0;
    }

    public int get(int key) {
        ListNode temp = cache.get(key);
        if(temp != null){
            moveToFoot(temp);
            return temp.value;
        }
        return -1;
    }

    public void put(int key, int value) {

        ListNode temp = cache.get(key);
        if(temp != null){
            temp.value = value;
            moveToFoot(temp);
        }else {
            if(cursor == capacity){
                ListNode delNode = popListNode();
                removeNode(delNode);
                cache.remove(delNode.key,delNode);
                cursor--;
            }
            temp = new ListNode(key,value);
            addNode(temp);
            cursor++;
        }
        cache.put(key,temp);
    }

    /**
     * 在尾部哨兵节点之前新增节点，同时将哨兵节点前指针指向的节点指向新插入的节点
     * @param node
     */
    private void addNode(ListNode node){

        node.next = this.footer;
        node.previous = this.footer.previous;

        this.footer.previous.next = node;
        this.footer.previous = node;

    }

    /**
     * 移除链表节点，由于有前后指针，时间复杂度为O(1)
     * @param node
     */
    private void removeNode(ListNode node){
        ListNode pre = node.previous;
        ListNode next = node.next;
        pre.next = next;
        next.previous = pre;
    }

    /**
     * 移动节点至链表尾部，即先将节点从原来的链表中删除，后做一次节点的插入操作
     * @param node
     */
    private void moveToFoot(ListNode node){
        this.removeNode(node);
        this.addNode(node);
    }

    /**
     * 由于存在哨兵节点，弹出当前节点只需要弹出头部哨兵节点后指针指向的节点即可
     * @return
     */
    private ListNode popListNode(){
        ListNode temp = header.next;
        removeNode(temp);
        return temp;
    }

}

```
