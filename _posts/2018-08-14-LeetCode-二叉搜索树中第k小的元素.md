---
layout:     post
title:      LeetCode二叉搜索树中第k小的元素
subtitle:   二叉搜索树中第k小的元素
date:       2018-08-14
author:     KaithyXu
header-img: /img/blogpicture.jpg
catalog: true
tags:
    - algorithm
---

## 二叉搜索树中第k小的元素

### 题目描述

给定一个二叉搜索树，编写一个函数来查找其中第k个最小的元素

### 解题思路

首先，根据题意，这是一颗二叉搜索树，其特性是对于树中的任何非叶子结点，其左子节点一定小于父节点，其右子节点一定大于父节点，而二叉搜索树的一个重要特性是，如果对二叉搜索树采取中序遍历，会得到一个递增的数组，所以这道题就变为找到第k个呗遍历到的节点

### 代码实现

```
public class TheMinKElement {

    int record = 0; //记录遍历节点个数
    int result = 0; //记录第k小的元素
    public int kthSmallest(TreeNode root, int k) {
         find(root,k);
        return result;
    }

    private void find(TreeNode root, int k){

        if(root == null){
            return ;
        }

        find(root.left,k);
        record++; // 中序遍历，每有一个节点，记录加1
        if(k == record){ //若遍历到第k个节点，就是要找的结果，记录下来后直接返回
            result = root.val;
            return ;
        }
        find(root.right,k);
        return ;
    }
}

```
