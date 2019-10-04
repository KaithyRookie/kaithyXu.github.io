---
layout:     post
title:      LeetCode数组中的第K个最大元素
subtitle:   数组中的第K个最大元素
date:       2018-10-04
author:     KaithyXu
header-img: img/reverseWords.jpg
catalog: true
tags:
    - algorithm
---
## 数组中的第K个最大元素


### 题目描述

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

示例 1:

输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
示例 2:

输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4
说明:

你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。

### 解题思路

 利用快排算法先对数组进行排序，然后再根据数组下标取得第K大的元素，其中，在快排递归的时候，先判断数组中间位置与k的大小，根据比较结果选择递归的方向以减少不必要的递归消耗

### 代码实现

```
class Solution {
    public int findKthLargest(int[] nums, int k){
        if(nums.length < k){
            return 0;
        }
        quickSort(nums,0,nums.length-1,k);

        return nums[k-1];
    }

    private void quickSort(int[] nums,int left,int right,int k){

        int center = (left+right)/2;
        if(left >= right){
            return;
        }
        int pivot = chosePivot(nums,left,right,center);

        int i = left+1;
        int j= right-1;
        while (i < j){
            while (nums[i] >= nums[center] && i < center){ i++;}
            while (nums[j] < nums[center] && j > center){j--; }

            if(nums[i] <= pivot && pivot <= nums[j]){
                swap(nums,i,j);
                if(i == center){
                    center = j;
                }else if(j == center){
                    center = i;
                }
            }
        }
        if(k <= center){
            quickSort(nums,left,center,k);
        }else if(k > center){
            quickSort(nums,center+1,right,k);
        }

    }

    private int chosePivot(int[] nums,int left,int right,int center){
        if(nums[right] > nums[center]){
            swap(nums,right,center);

        }
        if(nums[left] < nums[right]){
            swap(nums,left,right);

        }

        if(nums[left] < nums[center]){
            swap(nums,left,center);
        }


        return nums[center];

    }

    private void swap(int[] nums,int left,int right){
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
    }
}

```

