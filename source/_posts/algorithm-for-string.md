---
title: 算法学习笔记 — 数字与字符串处理
author: 子丶言
date: 2020-09-07 17:15:46
mathjax: true
tags: ['算法', '数字与字符串处理', '学习笔记']
categories: ['算法学习笔记']
---

算法并不是什么高深的领域内容，我们平时在项目开发中几乎都有涉及，比如翻转字符串、字符串截取，这些熟悉的函数方法你一定有所接触。
<!-- more -->

## 翻转整数

> 给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

    示例 1:
    输入: 123
    输出: 321

    示例 2:
    输入: -123
    输出: -321

    示例 3:
    输入: 120
    输出: 21

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 $ \begin{bmatrix} -2^{31}, 2^{31} - 1 \end{bmatrix} $。请根据这个假设，如果反转后整数溢出那么就返回 0。

### 解法一：利用数组的 reverse 方法

思路：在 JavaScript 中数组有一个 reverse 方法，该方法是将数组元素进行翻转。既然数组有翻转的方法，那么我们能不能想办法将数字转换成数组呢？目前虽然没有直接将数字转换成数组的方法，但我们可以考虑先将数字变成字符串，然后将字符串数值转换数组，利用数组的 reverse 方法翻转数组后，将转换回数字。

```js
/**
 * 翻转整数
 * @param {Number} num 需要翻转的整数
 * @return {Number} 翻转后的整数
 */
function reverseInteger(num) {
  // 异常值处理
  if (typeof num !== 'number') return NaN
  // 获取绝对值状态下的翻转整数
  let result = parseInt(Math.abs(num).toString().split('').reverse().join(''), 10)
  // 补充符号
  result = num < 0 ? 0 - result : result
  // 判断溢出
  if (result >= Math.pow(2, 31) - 1 || result <= Math.pow(-2, 31) + 1) return 0
  return result
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  当前算法将数值转换为字符串进行字符串转数组，数组翻转以及数组转字符串操作，该过程时间消耗与数值的位数 $ n $ 有关，可以认为是 $ 3n $ 的时间消耗，忽略系数，因此时间复杂度可以认为是 $ O(n) $。
  考虑到该题限制了数值范围为 32 位的有符号整数，其最大整数位长度为 `11`，也可以认为是常数时间复杂度 $ O(1) $。

- 空间复杂度：$ O(n) $
  当前算法在转换过程中临时创建了字符串和数组对象，临时空间的大小与数值的位数 $ n $ 有关，因此空间复杂度为 $ O(n) $。
  考虑到该题限制了数值范围为 32 位的有符号整数，其最大整数位长度为 `11`，也可以认为是常数空间复杂度 $ O(1) $。

### 解法二：借助欧几里得算法求解

思路：我们借鉴欧几里得求最大公约数的方法来解题。符号的处理逻辑不变，对于整数部分我们通过模 `10` 取到最低位，然后又通过乘 `10` 将最低位迭代到最高位，完成数值翻转。

```js
/**
 * 翻转整数
 * @param {Number} num 需要翻转的整数
 * @return {Number} 翻转后的整数
 */
function reverseInteger(num) {
  // 异常值处理
  if (typeof num !== 'number') return NaN
  // 获取相应数的绝对值
  let int = Math.abs(num)
  let result = 0
  // 遍历循环生成每一位数字
  while (int !== 0) {
    // 借鉴欧几里得算法，从 num 的最后一位开始取值拼成新的数
    result = int % 10 + result * 10
    // 剔除掉被消费的部分
    int = Math.floor(int / 10)
  }
  // 补充符号
  result = num < 0 ? 0 - result : result
  // 判断溢出
  if (result >= Math.pow(2, 31) - 1 || result <= Math.pow(-2, 31) + 1) return 0
  return result
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  由于当前算法只使用了 `while` 循环，循环次数为 $ n $ 次，即数值的整数长度，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  当前算法只用到了常数个变量，因此空间复杂度为 $ O(1) $。


## 有效的字母异位词

> 给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

    示例 1:
    输入: s = "anagram", t = "nagaram"
    输出: true

    示例 2:
    输入: s = "rat", t = "car"
    输出: false

### 解法一：利用数组的 sort 方法

思路：先将字符串转为数组，利用数组的 sort 方法进行默认排序，将排序后的数组转会字符串，比较字符串是否相等。

```js
/**
 * 判断是否为有效的字母异位词
 * @param {String} source 当前字符串
 * @param {String} target 目标字符串
 * @return {Boolean} 排序后的字符串
 */
function isAnagram(source, target) {
  if (typeof source !== 'string' || typeof target !== 'string') {
    return false
  }
  /**
   * 字符串排序
   * @param {String} str 需要排序的字符串
   * @return {String} 排序后的字符串
   */
  const sortString = str => {
    return str.split('').sort().join('')
  }
  return sortString(source) === sortString(target)
}
```

#### 复杂度分析

- 时间复杂度：$ O(nlog(n)) $
  当前算法主要借助了数组的 `sort` 方法，但 JavaScript 中 `sort` 方法的实现原理，当数组长度小于等于 10 的时候，采用插入排序 $ O(n^2) $，大于 10 的时候，采用快速排序，快速排序的平均时间复杂度是 $ O(nlog(n)) $。

- 空间复杂度：$ O(n) $
  算法中申请了 2 个数组变量用于存放字符串分割后的字符串数组，所以数组空间长度跟字符串长度线性相关，所以为 $ O(n) $。
