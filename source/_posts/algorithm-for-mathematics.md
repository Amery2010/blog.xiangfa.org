---
title: 算法学习笔记 — 数学领域
author: 子丶言
date: 2020-10-22 17:29:32
mathjax: true
tags: ['算法', '数学', '学习笔记']
categories: ['算法学习笔记']
---

在做算法时，可能需要运用一些数学知识。数学也是计算机的基础，但大部分算法只涉及到高中以下的知识，因此不需要过于担心能否看懂题目。
<!-- more -->

## 罗马数字转整数

> 罗马数字包含以下七种字符：I， V， X， L，C，D 和 M。
> 分别对应的数值为：1 ，5，10，50，100，500，1000 。
> 例如， 罗马数字 3 写做 III，即为三个并列的 1。12 写做 XII，即为 X+II。 26 写做 XXVII, 即为 XX+V+I。
> 通常情况下，不能出现超过连续三个相同的罗马数字并且罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

    I 可以放在 V(5) 和 X(10) 的左边，来表示 4 和 9。
    X 可以放在 L(50) 和 C(100) 的左边，来表示 40 和90。
    C 可以放在 D(500) 和 M(1000) 的左边，来表示 400 和 900。

> 给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

    输入:"III"
    输出:3

    输入:"IV"
    输出:4

    输入:"LVIII"
    输出:58

    输入:"MCMXCIV"
    输出:1994

详解：[LeetCode 罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/solution/yong-shi-9993nei-cun-9873jian-dan-jie-fa-by-donesp/)

### 解法一：遍历

思路：罗马数字的特殊值组合主要有 IV、VI、IX、XI 等，对于 IV，如果用右边的值减去左边的值可以得到 `5 - 1 = 4`，而对于 VI，如果使用右边的值加上左边的值可以得到 `5 + 1 = 6`。因此我们在遍历过程中每次取前后两个值，就可以得到如果左侧的值小于右侧的值，则使用大值减小值，如果左边的值大于右边的值，则大值加小值的方案。

```js
/**
 * 罗马数字转整数
 * @param {string} str 需要转换的罗马数字
 * @return {number} 转换后的整数
 */
function romanToInteger(str) {
  // 异常值处理
  if (typeof str !== 'string') return NaN
  const getValue = char => {
    switch (char) {
      case 'I': return 1
      case 'V': return 5
      case 'X': return 10
      case 'L': return 50
      case 'C': return 100
      case 'D': return 500
      case 'M': return 1000
      default: return 0
    }
  }
  let sum = 0
  // 获取起始值
  let preValue = 0
  for (let i = 0; i < str.length; i++) {
    // 当前值
    const curValue = getValue(str.charAt(i))
    if (preValue < curValue) {
      sum -= preValue
    } else {
      sum += preValue
    }
    preValue = curValue
  }
  sum += preValue
  return sum
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法只使用了一次遍历，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法只使用了一些常规值，因此空间复杂度为 $ O(1) $。
