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

详解：[LeetCode 整数反转](https://leetcode-cn.com/problems/reverse-integer/solution/zheng-shu-fan-zhuan-by-leetcode/)

### 解法一：利用数组的 reverse 方法

思路：在 JavaScript 中数组有一个 reverse 方法，该方法是将数组元素进行翻转。既然数组有翻转的方法，那么我们能不能想办法将数字转换成数组呢？目前虽然没有直接将数字转换成数组的方法，但我们可以考虑先将数字变成字符串，然后将字符串数值转换数组，利用数组的 reverse 方法翻转数组后，将转换回数字。

```js
/**
 * 翻转整数
 * @param {number} num 需要翻转的整数
 * @return {number} 翻转后的整数
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
 * @param {number} num 需要翻转的整数
 * @return {number} 翻转后的整数
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

详解：[LeetCode 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/solution/you-xiao-de-zi-mu-yi-wei-ci-by-leetcode/)

### 解法一：利用数组的 sort 方法

思路：先将字符串转为数组，利用数组的 sort 方法进行默认排序，将排序后的数组转会字符串，比较字符串是否相等。

```js
/**
 * 判断是否为有效的字母异位词
 * @param {string} source 当前字符串
 * @param {string} target 目标字符串
 * @return {boolean} 排序后的字符串
 */
function isAnagram(source, target) {
  if (typeof source !== 'string' || typeof target !== 'string') {
    return false
  }
  /**
   * 字符串排序
   * @param {string} str 需要排序的字符串
   * @return {string} 排序后的字符串
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


### 解法二：计数累加法

思路：先统计第一个字符串的字符类型和出现次数，然后对另一个字符串进行对比统计，如果字符类型不一致或者出现次数不同，则表示两个字符串不相等。

```js
/**
 * 判断是否为有效的字母异位词
 * @param {string} source 当前字符串
 * @param {string} target 目标字符串
 * @return {boolean} 排序后的字符串
 */
function isAnagram(source, target) {
  if (typeof source !== 'string' || typeof target !== 'string') {
    return false
  }
  if (source.length !== target.length) {
    return false
  }
  const hash = Object.create(null)
  for (const k of source) {
    hash[k] = k in hash ? hash[k] + 1 : 1
  }
  for (const k of target) {
    if (!(k in hash)) return false
    hash[k] -= 1
  }
  return true
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  当前算法只使用了两次单循环，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  当前算法只使用 `hash` 和 `k` 两个变量，空间大小不随字符串的变量而变化。


## 字符串中的第一个唯一字符

> 给定一个字符串，假定该字符串只包含小写字母，找到它的第一个不重复的字符，并返回它的索引。如果不存在，则返回 -1。

    s = "leetcode"
    返回 0.

    s = "loveleetcode",
    返回 2.

详解：[LeetCode 字符串中的第一个唯一字符](https://leetcode-cn.com/problems/first-unique-character-in-a-string/solution/zi-fu-chuan-zhong-de-di-yi-ge-wei-yi-zi-fu-by-leet/)

### 解法一：利用 js 自带方法求解

思路：如果字符串某个字符的正向索引值和反向索引值相同，则表示该字符只出现了一次。

```js
/**
 * 获取字符串中的第一个唯一字符
 * @param {string} str 目标字符串
 * @return {number} 唯一字符的索引值，如果不存在，则返回 -1。
 */
function firstUniqChar(str) {
  if (typeof str !== 'string') return -1
  for (let i = 0; i < str.length; i += 1) {
    if (str.indexOf(str[i]) === str.lastIndexOf(str[i])) {
      return i
    }
  }
  return -1
}
```

#### 复杂度分析

- 时间复杂度：$ O(n^2) $
  循环的时间复杂度为 $ O(n) $，`indexOf` 与 `lastIndexOf` 的时间复杂度均为 $ O(n) $，所以总时间复杂度应该为 $ O(n^2) $。

- 空间复杂度：$ O(1) $
  除了临时变量 `i`，没有开辟额外的存储空间。

### 解法二：利用哈希

思路：先使用一个对象存储所有的字符出现次数，再找出对象中字符只出现一次的下标。

```js
/**
 * 获取字符串中的第一个唯一字符
 * @param {string} str 目标字符串
 * @return {number} 唯一字符的索引值，如果不存在，则返回 -1。
 */
function firstUniqChar(str) {
  if (typeof str !== 'string') return -1
  const hash = Object.create(null)
  for (const k of str) {
    hash[k] = k in hash ? hash[k] + 1 : 1
  }
  for (let i = 0; i < str.length; i += 1) {
    if (hash[str[i]] === 1) {
      return i
    }
  }
  return -1
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法存在两次遍历，每次遍历的时间复杂度为 $ O(n) $，因为不存在嵌套遍历，因此时间复杂度只与变量 `str` 有关。

- 空间复杂度：$ O(1) $
  当前算法只使用 `hash`、`k` 和 `i` 三个变量，空间大小不随字符串的变量而变化。


## 验证回文串

> 给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。本题中，我们将空字符串定义为有效的回文串。

    示例 1:
    输入: "A man, a plan, a canal: Panama"
    输出: true

    示例 2:
    输入: "race a car"
    输出: false

详解：[LeetCode 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/solution/yan-zheng-hui-wen-chuan-by-leetcode-solution/)

### 解法一：字符串遍历

思路：先移除字符串中的非字母和数字，再将字符串转换为数组，再对数组首尾一一比较，即可得出结果。

```js
/**
 * 验证回文串
 * @param {string} str 需要判断的字符串
 * @return {boolean} 是否为回文串
 */
function isPalindrome(str) {
  if (typeof str !== 'string') return false
  // 将传入的字符串,统一转化为小写,同时去除非字母和数字,在转换为数组
  const strArr = str.toLowerCase().replace(/[^a-z0-9]/g, '').split('')
  let i = 0
  let j = strArr.length - 1
  // 循环比较元素
  while (i < j) {
    // 从首尾开始, 一一比较元素是否相等
    if (strArr[i] === strArr[j]) {
      // 若相等,即第二个元素和倒数第二个元素继续比较,依次类推
      i += 1
      j -= 1
    } else {
      // 只要有一个相对位置上不相等,既不是回文串
      return false
    }
  }
  return true
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法中 while 循环最多执行 $ n/2 $ 次，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(n) $
  该算法使用了长度为 $ n $ 的数组，因此空间复杂度为 $ O(n) $。

### 解法二：利用数组的 reverse 方法

思路：先移除字符串中的非字母和数字，然后利用数组的 reverse 方法将字符串翻转，再和原字符串进行比较，即可得到结果。

```js
/**
 * 验证回文串
 * @param {string} str 需要判断的字符串
 * @return {boolean} 是否为回文串
 */
function isPalindrome(str) {
  if (typeof str !== 'string') return false
  // 将传入的字符串,统一转化为小写,同时去除非字母和数字,在转换为数组
  const strArr = str.toLowerCase().replace(/[^a-z0-9]/g, '').split('')
  // 将2个字符进行比较得出结果
  return strArr.join('') === strArr.reverse().join('')
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法中所用的 js 方法的时间复杂度都为 $ O(n) $，且都在独立的循环中执行，因此，总的时间复杂度依然为 $ O(n) $。

- 空间复杂度：$ O(n) $
  该算法使用了长度为 $ n $ 的数组，因此空间复杂度为 $ O(n) $。


## 最长回文子串

> 给定一个字符串 str，找到 str 中最长的回文子串。你可以假设 str 的最大长度为 1000。

    示例
    输入: "babad"
    输出: "bab"
    注意: "aba" 也是一个有效答案。

详解：[LeetCode 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/solution/zui-chang-hui-wen-zi-chuan-by-leetcode-solution/)

### 解法一：动态规划法

动态规划的思想，是希望把问题划分成相关联的子问题；然后从最基本的子问题出发来推导较大的子问题，直到所有的子问题都解决。  
根据字符串的长度，建立一个矩阵 dp, 通过不同情况的判断条件，通过 `dp[i][j]` 表示 `s[i]` 至 `s[j]` 所代表的子串是否是回文子串。

```js
/**
 * 获得最长回文子串
 * @param {string} str 需要判断的字符串
 * @return {string} 最长回文串
 */
function longestPalindrome(str) {
  if (typeof str !== 'string') return ''
  const dp = []
  for (let i = 0; i < str.length; i++) {
    dp[i] = []
  }
  let max = -1
  let result = ''
  for (let l = 0; l < str.length; l++) {
    // l为所遍历的子串长度 -1，即左下标到右下标的长度
    for (let i = 0; i + l < str.length; i++) {
      const j = i + l
      // i为子串开始的左下标，j为子串开始的右下标
      if (l === 0) {
        // 当子串长度为1时，必定是回文子串
        dp[i][j] = true
      } else if (l <= 2) {
        // 长度为2或3时，首尾字符相同则是回文子串
        if (str[i] === str[j]) {
          dp[i][j] = true
        } else {
          dp[i][j] = false
        }
      } else {
        // 长度大于3时，若首尾字符相同且去掉首尾之后的子串仍为回文，则为回文子串
        if ((str[i] === str[j]) && dp[i + 1][j - 1]) {
          dp[i][j] = true
        } else {
          dp[i][j] = false
        }
      }
      if (dp[i][j] && l > max) {
        max = l
        result = str.substring(i, j + 1)
      }
    }
  }
  return result
}
```

#### 复杂度分析

- 时间复杂度：$ O(n^2) $
  该算法存在两层遍历，因此时间复杂度为 $ O(n^2) $。

- 空间复杂度：$ O(n) $
  该算法申请了一个长度为 `n` 的数组用于结果存储，因此空间复杂度为 $ O(n) $。

### 解法二：中心扩展

思路：回文子串一定是对称的，所以我们可以每次选择一个中心，然后从中心向两边扩展判断左右字符是否相等。  
中心点的选取有两种情况：  
- 当长度为奇数时，以单个字符为中心。
- 当长度为偶数时，以两个字符之间的空隙为中心。

```js
/**
 * 获得最长回文子串
 * @param {string} str 需要判断的字符串
 * @return {string} 最长回文串
 */
function longestPalindrome(str) {
  if (typeof str !== 'string') return ''
  if (str.length < 1) return ''
  const expandFromCenter = (str, left, right) => {
    while (left >= 0 && right < str.length && str[left] === str[right]) {
      left -= 1
      right += 1
    }
    return right - left - 1
  }
  let start = 0
  let end = 0
  for (let i = 0; i < str.length; i++) {
    // 中心的两种选取（奇对称和偶对称）
    const len1 = expandFromCenter(str, i, i)
    const len2 = expandFromCenter(str, i, i + 1)
    // 两种组合取最大的回文子串长度
    const len = Math.max(len1, len2)
    // 如果此位置为中心的回文数长度大于之前的长度，则进行处理
    if (len > end - start) {
      start = i - Math.floor((len - 1) / 2)
      end = i + Math.floor(len / 2)
    }
  }
  return str.substring(start, end + 1)
}
```

#### 复杂度分析

- 时间复杂度：$ O(n^2) $
  该算法存在两层循环嵌套，因此遍历的最大次数为 $ n^2 $。

- 空间复杂度：$ O(1) $
  该算法只使用了常量，因此空间复杂度为 $ O(1) $。
