---
title: leetcode-1 两数之和
date: 2021-06-19 16:42:44
tags: leetcode
categories: algoririthms
---
## 题目描述
给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那**两个**整数，并返回它们的数组下标。你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。你可以按任意顺序返回答案。
<!-- more -->
示例 1：
```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```
示例 2：
```
输入：nums = [3,2,4], target = 6
输出：[1,2]
```
示例 3：
```
输入：nums = [3,3], target = 6
输出：[0,1]
```

## 1 暴力求解
此方法很简单，遍历每个元素，并查找是否存在两个值的和与target相等。
#### code
``` golang
func twoSum(nums []int, target int) []int {
    for i := range nums {
        for j := i + 1; j < len(nums); j++ {
            if nums[i] + nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}
```
#### 分析
* 时间复杂度：O(n<sup>2</sup>)
* 空间复杂度：O(1)

## 2 hash表
### 2.1 两遍hash表
为了对运行时间进行优化，我们需要一种更有效的方式来检查数组中是否存在目标元素，如果存在，如果存在，需要找到其索引。保持数组中每个元素与其索引之间最好的方式就是hash表。

步骤：
1. 首先我们遍历数组按照(key-value)—>(nums[i]-i)的关系存入hash表中；
2. 然后遍历数组，找寻表中map[target-nums[i]]的下标值；
以上方法需要两次遍历数组。

#### code
``` golang
func twoSum(nums []int, target int) []int {
    hashMap := make(map[int]int, len(nums))
    for i := range nums {
    	hashMap[nums[i]] = i
    }
    for i := range nums {
        if j, ok := hashMap[target - nums[i]]; ok && j != i {
            return []int{i, j}
        }
    }
    return nil
}
```
#### 分析
* 时间复杂度：O(n)，虽然两次遍历数组，但是其时间复杂度依然为O(n)
* 空间复杂度：O(n)，所需额外的空间存储hash表中的数据，且必定为O(n)

### 2.2 一遍hash表
事实上，我们可以一次遍历完成以上工作，即在第一次遍历时即回头check是否存在和当前元素和为target的元素，不过需要注意的是，此时找到的元素都是之前插入的，所以返回时的i,j顺序需要颠倒一下。

#### code
``` golang
func twoSum(nums []int, target int) []int {
    hashMap := make(map[int]int, len(nums))
    for i := range nums {
        if j, ok := hashMap[target - nums[i]]; ok {
            return []int{j, i}
        }
        hashMap[nums[i]] = i
    }
    return nil
}
```
#### 分析
* 时间复杂度：O(n)，虽然时间复杂度和前一种方法一样，但是其下限比较低，如果元素比较靠前，可以很快找到
* 空间复杂度：O(n)，虽然空间复杂度和前一种方法一样，但是其下限较低，最大需要存储n个元素，而前一题是必然存储n个元素，故而必定为O(n)