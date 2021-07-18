---
title: leetcode-3 无重复字符的最长子串
date: 2021-06-20 15:20:26
tags: leetcode
categories: algoririthms
---
## 题目描述
给定一个字符串，请你找出其中不含有重复字符的**最长子串**的长度。
<!-- more -->
示例 1:
```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```
示例 2:
```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```
示例 3:
```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```
示例 4:
```
输入: s = ""
输出: 0
```

**提示**：
* 0 <= s.length <= 5 * 104
* s 由英文字母、数字、符号和空格组成

## 暴力求解
暴力求解的分析很简单，即求解所有无重复字符的字串，找出最长的那个。
#### code
``` golang
func lengthOfLongestSubstring(s string) int {
    l := len(s)
    if l <= 1 {
        return l
    }
    maxLen := 1
    bs := []byte(s)
    for i := 0; i < l -1; i++ {
        ll := 1
    loop:
        for j := i + 1; j < l; j++ {
            for k := i; k < j; k++ {
                if bs[k] == bs[j] {
                    break loop
                }
            }
            ll++
            if ll > maxLen {
                maxLen = ll
            }
        }
    }
    return maxLen
}
```
#### 分析
* 时间复杂度：O(n<sup>3</sup>)
* 空间复杂度：O(1)

## 滑动窗口
如果我们选择字符串中第k个字符作为起始位置，并且得到了不包含重复字符的最长字串的结束位置为rk。那么当我们选择第k+1个字符作为起始位置时，首先从k+1到rk的字符就是不重复的，且少了第k个字符，我们可以尝试继续增大rk，直到右边出现重复字符为止。那么，我们可以用“滑动窗口”解决这个问题：
1. 我们使用两个指针表示字符串中的某个子串（或窗口）的左右边界，其中左指针代表着上文中起始位置k，而右指针即为上文中的rk；​
2. 每一步操作中，我们将左指针右移一格，表示我们开始枚举下一个字符作为起始位置；然后我们可以不断地向右移动右指针，但需要保证这两个指针对应的子串中没有重复的字符。在移动结束后，这个子串就对应着 以左指针开始的，不包含重复字符的最长子串。我们记录下这个子串的长度；
3. 枚举结束后，这个字串的长度记为最长字串。
4. 判断数据结构是否有重复字符，选用hash表。在左指针向右移动的时候，我们从哈希集合中移除一个字符，在右指针向右移动的时候，我们往哈希集合中添加一个字符。

#### code
``` golang
func lengthOfLongestSubstring(s string) int {
    l := len(s)
    if l <= 1 {
        return l
    }
    sByte := []byte(s)
    hashMap := make(map[byte]int, l)
    maxLen := 0
    start := 0
    for i:=0; i<l;i++ {
        if _, ok := hashMap[sByte[i]]; ok {
        	// 如果找到了重复的，删除hash表头，重置i(也就是rk)，右移start(也就是k)
            delete(hashMap, sByte[start])
            start++
            i--
            continue
        }
        if i - start + 1 > maxLen {
            maxLen = i - start + 1
        }
        hashMap[sByte[i]] = i
    }
    return maxLen
}
```

#### 分析
* 时间复杂度：O(n)
* 空间复杂度：O(∣Σ∣)，可以默认为所有 ASCII 码在 [0, 128)[0,128) 内的字符，即 |\Sigma| = 128∣Σ∣=128

## 滑动窗口优化
上一种方法是官方给出的标准解答，但是我认为这个答案的左指针每次只移动一次，这一步会导致很多无用的循环。我们分析可以发现，重复字符不一定是Set最早添加的字符，可能还要很多次循环才能到达，这些都是无效循环，不如直接跳转。即每一步操作中，我们可以不断地向右移动右指针，并更新hash表，但需要保证这两个指针对应的子串中没有重复的字符。在移动结束后，这个子串就对应着以左指针开始的，不包含重复字符的最长子串，我们记录下这个子串的长度。如果出现重复字符，且字符在左指针的右边，那我们直接跳到重复字符（前一个）的右边继续循环即可。

#### code
``` golang
func lengthOfLongestSubstring(s string) int {
    l := len(s)
    if l <= 1 {
        return l
    }
    sByte := []byte(s)
    hashMap := make(map[byte]int, l)
    maxLen := 0		// 最大长度
    start := 0		// 左指针
    for i := range sByte {
    	// 如果出现重复，且此重复字符在左指针的右边，我们直接移到其后去
        if j, ok := hashMap[sByte[i]]; ok {
            if j + 1 > start {
                start = j + 1
            }
        }
        if i - start + 1 > maxLen {
            maxLen = i - start + 1
        }
        hashMap[sByte[i]] = i
    }
    return maxLen
}
```
#### 分析
* 时间复杂度：O(n)，虽然有着相同的时间复杂度，但是此方法比上一个方法更加简洁，耗时也更少。
* 空间复杂度：O(∣Σ∣)