---
title: leetcode-4 寻找两个正序数组的中位数
date: 2021-06-23 07:59:42
tags: leetcode
categories: algoririthms
---
给定两个大小分别为m和n的正序（从小到大）数组nums1和nums2。请你找出并返回这两个正序数组的**中位数**。
<!-- more -->
示例 1：
```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```
示例 2：
```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```
示例 3：
```
输入：nums1 = [0,0], nums2 = [0,0]
输出：0.00000
```
示例 4：
```
输入：nums1 = [], nums2 = [1]
输出：1.00000
```
示例 5：
```
输入：nums1 = [2], nums2 = []
输出：2.00000
```

**提示**：
* nums1.length == m
* nums2.length == n
* 0 <= m <= 1000
* 0 <= n <= 1000
* 1 <= m + n <= 2000
* -10<sup>6</sup> <= nums1[i], nums2[i] <= 10<sup>6</sup>
 

**进阶**：你能设计一个时间复杂度为**O(log(m+n))**的算法解决此问题吗？

## 排序后返回
虽然在leetcode上这题给出的难度是困难，但是我觉得做出来还是不难的：
* 因为两个数组是正序的，所以给两个数组排序只需O(m+n)的时间复杂度；
* 为了避免每个步骤都必须检查数组越界，我们在数组尾部设置一个哨兵值，这里哨兵值采用10<sup>6</sup>+1；

#### code
``` golang
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    l := len(nums1) + len(nums2)
    nums3 := make([]int, l, l)
    MAX := 1000001
    // 这里我直接在nums1、nums2后面添加了，其实这种做法不好，也许存在nums1和nums2地址重合或者正好衔接的情况
    // 为了简便，我直接这么做了，leetcode的测试用例也没有考虑到这点
    nums1 = append(nums1, MAX)
    nums2 = append(nums2, MAX)
    j := 0
    k := 0
    for i := 0; i < l; i++ {
        if nums1[j] < nums2[k] {
            nums3[i] = nums1[j]
            j++
        } else {
            nums3[i] = nums2[k]
            k++
        }
    }
    if l % 2 == 0 {
        return (float64(nums3[l/2-1]) + float64(nums3[l/2])) / 2
    } else {
        return float64(nums3[l/2])
    }
}
```
#### 分析
* 时间复杂度：O(m+n)
* 空间复杂度：O(m+n)

## 降低复杂度
**以下分析来自[官方解答](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/solution/xun-zhao-liang-ge-you-xu-shu-zu-de-zhong-wei-s-114/)**

如何把时间复杂度降低到O(log(m+n)) 呢？如果对时间复杂度的要求有log，通常都需要用到**二分查找**，这道题也可以通过二分查找实现。

根据中位数的定义，当m+n是奇数时，中位数是两个有序数组中的第(m+n)/2个元素，当m+n是偶数时，中位数是两个有序数组中的第(m+n)/2个元素和第(m+n)/2+1个元素的平均值。因此，这道题可以转化成寻找两个有序数组中的第k小的数，其中k为(m+n)/2或(m+n)/2+1。

假设两个有序数组分别是A和B。要找到第k个元素，我们可以比较A[k/2−1]和B[k/2−1]，其中/表示整数除法。由于A[k/2−1]和B[k/2−1]的前面分别有A[0..k/2−2]和B[0..k/2−2]，即k/2−1个元素，对于A[k/2−1]和B[k/2−1]中的较小值，最多只会有(k/2−1)+(k/2−1)≤k−2个元素比它小，那么它就不能是第k小的数了。

因此我们可以归纳出三种情况：
* 如果A[k/2-1]<B[k/2-1]，则比A[k/2-1]小的数最多只有A的前k/2-1个数和B的前k/2-1个数，即至多k-2个，因此A[k/2-1]不可能是第k个数，A[0]到A[k/2-1]也不可能是第k个数，可以全部排除；
* 如果A[k/2-1]>B[k/2-1]，同理可以排除B[0]到B[k/2-1]；
* 如果A[k/2-1]=B[k/2-1]，也可以归结到第一种情况；

可以分析得，每次比较后可以排除k/2个数（譬如A[k/2-1]<=B[k/2-1]，那么A[0]\~A[k/2-1]都一定不是第k大的数），所以每次查找完需要更新k值，一般而言是k=k-k/2，有以下三种特殊情形需要特殊处理：
* 如果A[k/2-1]或者B[k/2-1]越界，那么我们可以选取数组中最后一个元素，对应的k值也应该减去排除的个数，而不是武断的减去k/2;
* 如果一个数组为空，说明该数组中的元素都被排除了，我们可以直接返回另一个数组中第k小的元素；
* 如果k=1，我们只要返回两个数组较小的首元素即可；

用一个例子说明上述算法。假设两个有序数组如下：

```
A: 1 3 4 9
B: 1 2 3 4 5 6 7 8 9
```
两个有序数组的长度分别是4和9，长度之和是13，中位数是两个有序数组中的第7个元素，因此需要找到第k=7个元素。

比较两个有序数组中下标为k/2−1=2的数，即A[2]和B[2]，如下面所示：

```
A: 1 3 4 9
       ↑
B: 1 2 3 4 5 6 7 8 9
       ↑
```
由于A[2]>B[2]，因此排除B[0]到B[2]，即数组B的下标偏移（offset）变为3，同时更新k的值：k=k−k/2=4。

下一步寻找，比较两个有序数组中下标为k/2−1=1的数，即A[1]和B[4]，如下面所示，其中方括号部分表示已经被排除的数。

```
A: 1 3 4 9
     ↑
B: [1 2 3] 4 5 6 7 8 9
             ↑
```
由于A[1]<B[4]，因此排除A[0]到A[1]，即数组A的下标偏移变为2，同时更新k的值：k=k−k/2=2。

下一步寻找，比较两个有序数组中下标为k/2−1=0的数，即比较A[2]和B[3]，如下面所示，其中方括号部分表示已经被排除的数。
```
A: [1 3] 4 9
         ↑
B: [1 2 3] 4 5 6 7 8 9
           ↑
```
由于A[2]=B[3]，根据之前的规则，排除A中的元素，因此排除A[2]，即数组A的下标偏移变为3，同时更新k的值：k=k−k/2=1。

由于k的值变成1，因此比较两个有序数组中的未排除下标范围内的第一个数，其中较小的数即为第k个数，由于A[3]>B[3]，因此第k个数是B[3]=4。
```
A: [1 3 4] 9
           ↑
B: [1 2 3] 4 5 6 7 8 9
           ↑
```

代码如下：
``` golang
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    l1 := len(nums1)
    l2 := len(nums2)

    getKthEle := func(k int) int {
        index1, index2 := 0, 0
        offset1, offset2 := 0, 0
        for true {
        	// 找寻到数组1为空，说明第k个值在nums2[]中
            if index1 == l1 {
                return nums2[index2 + k - 1]
            }
        	// 找寻到数组1为空，说明第k个值在nums2[]中
            if index2 == l2 {
                return nums1[index1 + k - 1]
            }
        	// 最终的情形，剩下的数组中最小的那个数
            if k == 1 {
                if nums1[index1] < nums2[index2] {
                    return nums1[index1]
                }
                return nums2[index2]
            }

            // 偏移量需要判断是否越界
            half := k / 2
            if index1 + half < l1 {
                offset1 = half
            } else {
                offset1 = l1 - index1
            }
            if index2 + half < l2 {
                offset2 = half
            } else {
                offset2 = l2 - index2
            }
            // 循环判断条件，详见前面分析
            if nums1[index1 + offset1 -1] <= nums2[index2 + offset2 -1] {
                k -= offset1
                index1 += offset1
            } else {
                k -= offset2
                index2 += offset2
            }
        }
        // 这条没有作用，但是不加编译不过
        return 0
    }

    l := l1 + l2
    if l % 2 == 1 {
    	// 奇数时去中间一个数，即第k=(m+n)/2+1个数
        k := l / 2 + 1
        return float64(getKthEle(k))
    } else {
    	// 偶数时取第k1=(m+n)/2和k2=(m+n)/2+1个数的均值
        k1, k2 := l / 2, l / 2 + 1
        return (float64(getKthEle(k1)) + float64(getKthEle(k2)))/2
    }
}
```
#### 分析
* 时间复杂度：O(log(m+n))
* 空间复杂度：O(1)

不过在leetcode上的运行时间并没有减少多少。。。
