---
layout:     post
title:      LeetCode反转字符串中的单词 III
subtitle:   反转字符串中的单词 III
date:       2019-10-01
author:     KaithyXu
header-img: img/reverseWords.jpg
catalog: true
tags:
    - algorithm
---
## 反转字符串中的单词 III


### 题目描述

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

示例 1:

输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
注意：在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。

### 解题思路

 根据空格的ascii码来确认翻转的字符串，利用两个变量head与tail记录头尾的位置，当tail指向空格时，将head 至 tail-1之间的字符串通过ascii码的加减进行替换

### 代码实现

```
/**
 * 反转字符串中的单词 III
 * @author kaithy.xu
 * @date 2019-09-27 12:29
 */
public class ReverseWordsinAStringIII {

    public String reverseWords(String s) {

        char[] chars = s.toCharArray();
        int head = 0;
        for (int i = 0; i <= chars.length; i++) {

            if(i == chars.length || ' ' == chars[i] ) {
                reverseCharArray(chars,head, i-1);
                head = i+1;
            }

        }
        return String.valueOf(chars);
    }

    private void reverseCharArray(char[] chars, int head, int tail) {
        while (head < tail) {
            int diff = chars[tail] - chars[head];
            chars[head++] += diff;
            chars[tail--] -= diff;
        }
    }
}

```

