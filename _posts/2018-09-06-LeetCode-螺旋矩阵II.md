---
layout:     post
title:      LeetCode螺旋矩阵 II
subtitle:   螺旋矩阵 II
date:       2018-09-06
author:     KaithyXu
header-img: img/SpiralMatrix.jpg
catalog: true
tags:
    - algorithm
---
## 螺旋矩阵 II


### 题目描述

给定一个正整数 n，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

示例:

输入: 3
输出:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]


### 解题思路
解题思路：
在之前螺旋矩阵的基础上，只需要考虑矩阵为正方形，遍历四条边，有一个关键点就是确保遍历四条边的长度应该是对应相同的，利用一个全局变量，每次访问后改变量加一，直到访问结束。

### 代码实现

```
/**
 * 螺旋矩阵II
 * @author kaithy.xu
 * @date 2019-09-06 12:10
 */
public class SpiralMatrixII {

    private int element = 1;

    public int[][] generateMatrix(int n) {
        int[][] matrix = new int[n][n];
        int i=0,j=0;
        int col = n-1;
        int row = n-1;
        while (col >= i && row >= j) {
            if(j == col && i == row) {
                matrix[i][j] = element++;
            }else if(j != col && i != row) {
                addColElement(matrix,j,col-1,i);

                addRowElement(matrix,i,row-1,col);

                addColElement(matrix,col,j+1,row);

                addRowElement(matrix,row,i+1,j);
            }
            ++i;
            ++j;
            --col;
            --row;
        }
        return matrix;
    }

    /**
     * 固定行以遍历列
     * @param matrix
     * @param left
     * @param right
     * @param row
     */
    private void addColElement(int[][] matrix, int left , int right, int row) {
        if(left < right) {
            for (int i = left; i <= right; i++) {
                matrix[row][i] = element++;
            }
        }else if(left > right){
            for (int i = left; i >= right ; i--) {
                matrix[row][i] = element++;
            }
        }else {
            matrix[row][left] = element++;
        }

    }

    /**
     * 固定列以遍历行
     * @param matrix
     * @param head
     * @param tail
     * @param col
     */
    private void addRowElement(int[][] matrix, int head, int tail, int col) {
        if(head < tail) {
            for (int i = head; i <= tail; i++) {
                matrix[i][col] = element++;
            }
        }else if(head > tail){
            for (int i = head; i >= tail ; i--) {
                matrix[i][col] = element++;
            }
        }else {
            matrix[head][col] = element++;
        }
    }
    
}


```

