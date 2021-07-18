---
title: leetcode-5 最长回文子串
date: 2021-06-26 09:23:57
tags: leetcode
categories: algoririthms
---
给你一个字符串s，找到s中最长的回文子串。
<!-- more -->
示例 1：
```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```
示例 2：
```
输入：s = "cbbd"
输出："bb"
```
示例 3：
```
输入：s = "a"
输出："a"
```
示例 4：
```
输入：s = "ac"
输出："a"
```

**提示**：
1 <= s.length <= 1000
s 仅由数字和英文字母（大写和/或小写）组成

## 重心扩展
很容易分析，回文字符串有两种，分别为长度为奇数和偶数的，但是都符合从中心扩展的特点，代码如下：
``` golang
func longestPalindrome(s string) string {
    l := len(s)
    if l <= 1 {
        return s
    }
    
    sb := []byte(s)
    // 记录最大长度对应的坐标
    maxL, maxR := 0, 0
    for i := 0; i < l - 1; i++ {
    	// 第一种情形，偶数回文字符串
        tmpL1, tmpR1 := i, i
        if sb[i] == sb[i+1] {
            tmpR1++
            for offset := 1; i - offset >= 0 && i + offset + 1 < l; offset++ {
                if sb[i-offset] == sb[i+1+offset] {
                    tmpL1--
                    tmpR1++
                } else {
                    break
                }
            }
        }
        // 第二种情形，奇数回文字符串，这里不能在第一种情形的else中，需要避免连续三个或以上的奇数个相同字符串的情况被漏掉
        tmpL2, tmpR2 := i, i
        for offset := 1; i - offset >= 0 && i + offset < l; offset++ {
            if sb[i-offset] == sb[i+offset] {
                tmpL2--
                tmpR2++
            } else {
                break
            }   
        }
        
        // 判断最大长度是否需要更新
        if tmpR1 - tmpL1 > maxR - maxL {
            maxL, maxR = tmpL1, tmpR1
        }
        if tmpR2 - tmpL2 > maxR - maxL {
            maxL, maxR = tmpL2, tmpR2
        }
        // 很重要的一步，判断如果以第i个数或者i/i+1为中心的回文字符串的最大长度是否已经
        // 小于现有的最大长度，如果是的话，那继续循环下去没有意义，直接退出循环即可
        if l - i <= (maxR - maxL)/2 {
            break
        }
    }
    return string(sb[maxL:maxR+1])
}
```

#### 分析
* 时间复杂度：O(n<sup>2</sup>)，虽然时间复杂度看起来是这么多，但是内部循环首先达不到n级别，且最后的长度判断会避免很多不必要的循环，所以应该达不到O(n<sup>2</sup>)
* 空间复杂度：O(1)

## Manacher 算法
详见[官方解答——方法三](https://leetcode-cn.com/problems/longest-palindromic-substring/solution/zui-chang-hui-wen-zi-chuan-by-leetcode-solution/)。

但是跑下来的分数还没有上一种方法高，也是很奇怪。