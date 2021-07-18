---
title: leetcode-6 Z字形变换
date: 2021-06-26 15:22:40
tags: leetcode
categories: algoririthms
---
将一个给定字符串**s**根据给定的行数**numRows**，以从上往下、从左到右进行**Z**字形排列。
<!-- more -->
比如输入字符串为 "PAYPALISHIRING" 行数为 3 时，排列如下：
```
P   A   H   N
A P L S I I G
Y   I   R
```
之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："PAHNAPLSIIGYIR"。

请你实现这个将字符串进行指定行数变换的函数：

string convert(string s, int numRows);
 

示例 1：
```
输入：s = "PAYPALISHIRING", numRows = 3
输出："PAHNAPLSIIGYIR"
```
示例 2：
```
输入：s = "PAYPALISHIRING", numRows = 4
输出："PINALSIGYAHRPI"
解释：
P     I    N
A   L S  I G
Y A   H R
P     I
```
示例 3：
```
输入：s = "A", numRows = 1
输出："A"
```

**提示**：
1 <= s.length <= 1000
s 由英文字母（小写和大写）、',' 和 '.' 组成
1 <= numRows <= 1000

## 直接解法
我们可以先将重组后的字符分成n个子串，观察很好发现，若i为字符串下标（从0开始），`j = i % (2 \* numRows - 2)`，当j小于numRows时，此字符将正好是第j个子串的字符；当j大于等于numRows时，此时反过来，是第`(2 \* numRows - 2) - j`个字串的字符；顺序就正好按照原来的顺序即可，最后将字串按照顺序接上即可组成符合题意的字符串，详见代码。
#### code
``` golang
func convert(s string, numRows int) string {
    if numRows <= 1 {
        return s
    }
    ss := make([][]byte, numRows, numRows)
    rets := ""

    sb := []byte(s)
    divisor := 2 * numRows - 2
    for i := range sb {
        j := i % divisor
        if j < numRows {
            ss[j] = append(ss[j], sb[i])
        } else {
            ss[divisor - j] = append(ss[divisor - j], sb[i])
        }
    }
    for i := range ss {
        rets += string(ss[i])
    }
    return rets
}
```
#### 分析
* 时间复杂度：O(n)
* 空间复杂度：O(n)