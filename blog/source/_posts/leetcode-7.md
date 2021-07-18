---
title: leetcode-7 整数反转
date: 2021-06-26 16:45:23
tags: leetcode
categories: algoririthms
---
给你一个32位的有符号整数x，返回将x中的数字部分反转后的结果。如果反转后整数超过 32 位的有符号整数的范围[−2<sup>31</sup>,  2<sup>31</sup> − 1]，就返回 0。
**假设环境不允许存储 64 位整数（有符号或无符号）。**
<!-- more -->
示例 1：
```
输入：x = 123
输出：321
```
示例 2：
```
输入：x = -123
输出：-321
```
示例 3：
```
输入：x = 120
输出：21
```
示例 4：
```
输入：x = 0
输出：0
```

**提示**：

-231 <= x <= 231 - 1

## 直接解
因为golang的int是64位的，所以前面计算不用担心，结果反而是后面确定范围在[−2<sup>31</sup>,  2<sup>31</sup> − 1]让我错了好几次，真是笑死了。
#### code
``` golang
func reverse(x int) int {
    ret := 0
    xx := x
    for xx != 0 {
        yy := xx % 10
        ret = ret * 10 + yy
        xx = xx / 10
    }
    if ret > math.MaxInt32 || ret < math.MinInt32 {
        return 0
    }
    return ret
}
```
#### 分析
* 时间复杂度：O(log∣x∣)。翻转的次数即x十进制的位数
* 空间复杂度：O(1)