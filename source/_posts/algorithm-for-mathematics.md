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

### 解法一：模拟法

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


### 解法二：特殊值合并运算

思路：罗马数字的特殊逻辑中，除了 IV、IX、XL、XC、CD 和 CM 不符合累加原则以外，其他位数的数组都可以使用累加的形式进行取值。如果我们将特殊值作为一个“整体”来看待，这些特殊值也可以等于具体的值，即依然可以采用累加的原则。

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
      case 'IV': return 4
      case 'IX': return 9
      case 'XL': return 40
      case 'XC': return 90
      case 'CD': return 400
      case 'CM': return 900
      default: return 0
    }
  }
  const specChars = ['IV', 'IX', 'XL', 'XC', 'CD', 'CM']
  let sum = 0
  let i = 0
  while (i < str.length) {
    if (str.charAt(i + 1) !== ''
      && specChars.indexOf(str.charAt(i) + str.charAt(i + 1)) !== -1) {
      sum += getValue(str.charAt(i) + str.charAt(i + 1))
      i += 2
    } else {
      sum += getValue(str.charAt(i))
      i += 1
    }
  }
  return sum
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法只使用了一次遍历，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法只使用了一些常规值，因此空间复杂度为 $ O(1) $。


## Fizz Buzz

> 写一个程序，输出从 1 到 n 数字的字符串表示。
> 1.如果 n 是 3 的倍数，输出“Fizz”；
> 2.如果 n 是 5 的倍数，输出“Buzz”；
> 3.如果 n 同时是 3 和 5 的倍数，输出 “FizzBuzz”。

    示例
    n = 15
    返回:
    12Fizz4BuzzFizz78FizzBuzz11Fizz1314FizzBuzz
  
详解：[LeetCode 罗马数字转整数](https://leetcode-cn.com/problems/fizz-buzz/solution/fizz-buzz-by-leetcode/)

### 解法一：模拟法

思路：只需要判断 1 - n 的每个数字是否能被 3、5、15 整除，输出对应的字符串即可。

```js
/**
 * Fizz Buzz
 * @param {number} n 数字
 * @return {string} 结果字符串
 */
function fizzBuzz(n) {
  // 异常值处理
  if (typeof n !== 'number') return ''
  let result = ''
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) { // 被15整除
      result += 'FizzBuzz'
    } else if (i % 3 === 0) { // 被3整除
      result += 'Fizz'
    } else if (i % 5 === 0) { // 被5整除
      result += 'Buzz'
    } else {
      result += i.toString()
    }
  }
  return result
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法使用了一次数据遍历，循环次数依赖于数字 n，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法只使用了常规变量，因此时间复杂度为 $ O(1) $。


### 解法二：数组转对象

思路：将数据存储进数组，然后利用数组的 `Array.join()` 方法转换成字符串。

```js
/**
 * Fizz Buzz
 * @param {number} n 数字
 * @return {string} 结果字符串
 */
function fizzBuzz(n) {
  // 异常值处理
  if (typeof n !== 'number') return ''
  return Array.from(
    new Array(n),
    (t, i) => (t = (++i % 3 ? '' : 'Fizz') + (i % 5 ? '' : 'Buzz')) ? t : '' + i
  ).join('')
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法使用了 Array.from 的数组变量函数，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(n) $
  该算法中临时创建了长度为 `n` 的数组，因此空间复杂度为 $ O(n) $。


## 计数质数

> 统计所有小于非负整数 n 的质数的数量。其中 $ 0 <= n <= 5 * 10^6 $

    示例
    输入: 10
    输出: 4
    解释: 小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。

详解：[LeetCode 计数质数](https://leetcode-cn.com/problems/count-primes/solution/ji-shu-zhi-shu-bao-li-fa-ji-you-hua-shai-fa-ji-you/)

### 解法一：暴力求解

思路：求质数最简单的方案就是暴力求解，即穷举遍历。

```js
/**
 * 计数质数
 * @param {number} n 数字
 * @return {number} 质数数量
 */
function countPrimes(n) {
  // 异常值处理
  if (typeof n !== 'number') return 0
  let count = 0
  for (let i = 2; i < n; i++) {
    let sign = true
    for (let j = 2; j < i; j++) {
      if (i % j === 0) {
        sign = false
        break
      }
    }
    if (sign) count++
  }
  return count
}
```

#### 复杂度分析

- 时间复杂度：$ O(n^2) $
  该算法使用了两次遍历，时间复杂度为 $ O(n^2) $。

- 空间复杂度：$ O(1) $
  该算法只使用了常规的变量，因此空间复杂度为 $ O(1) $。


### 解法二：埃拉托斯特尼筛法

思路：从 2 开始，将每个质数的各个倍数，标记成合数。一个质数的各个倍数，是一个差为此质数本身的等差数列。此为这个筛法和试除法不同的关键之处，后者是以质数来测试每个待测数能否被整除。

```js
/**
 * 计数质数
 * @param {number} n 数字
 * @return {number} 质数数量
 */
function countPrimes(n) {
  // 异常值处理
  if (typeof n !== 'number') return 0
  const arr = new Array(n)
  let count = 0
  for (let i = 2; i < n; i++) {
    if (!arr[i - 1]) {
      count++
      for (let j = i * i; j <= n; j += i) {
        arr[j - 1] = true
      }
    }
  }
  return count
}
```

#### 复杂度分析

- 时间复杂度：$ O(nlog(log(n))) $
  对每一个 $ i $，要划掉 $ n/i $ 个数, 要进行 $ n/i $ 次运算，全部加起来，就是 $ n $ (从 1 到 $ \sqrt[]{n} $ 之间的 $ 1/i $ 之和)，简单讲就是 (从 1 到 $ n $ 之间的 $ 1/i $ 之和) 约等于 $ log $ (对所有 $ k $ 从 1 到 $ n $ 之间的 $ 1/k $ 之和)，后者是 $ log(n) $，所以前者就是 $ log(log(n)) $；最外层需要判断 $ n $ 次 ；所以最终时间复杂度为 $ O(nlog(log(n))) $。

- 空间复杂度：$ O(n) $
  该算法申请了一个长度为 n 的数组，因此空间复杂度为 $ O(n) $。


## 3 的幂

> 给定一个整数，写一个函数来判断它是否是 3 的幂次方。

    示例 1：
    输入: 27
    输出: true

    示例 2：
    输入: 0
    输出: false

    示例 3：
    输入: 9
    输出: true

    示例 4：
    输入: 45
    输出: false

**进阶**：你能不使用循环或者递归来完成本题吗？

详解：[LeetCode 3 的幂](https://leetcode-cn.com/problems/power-of-three/solution/3de-mi-by-leetcode/)

### 解法一：循环迭代

思路：只要让数字 `n`，循环除以 3，看能否被整除就可以了。

```js
/**
 * 求 3 的幂
 * @param {number} n 数字
 * @return {boolean} 是否为 3 的幂
 */
function isPowerOfThree(n) {
  // 异常值处理
  if (typeof n !== 'number') return false
  if (n < 1) return false
  while (n > 1) {
    // 如果该数字不能被 3 整除，则直接输出 false
    if (n % 3 !== 0) {
      return false
    } else {
      n = n / 3
    }
  }
  return true
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法只使用了一次遍历，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法未使用额外的内存空间，因此空间复杂度为 $ O(1) $。


### 解法二：整数限制

思路：常规的整数定义为 [-2147483648, 2147483647]，而在此范围内 3 的最大幂是 19，即 Math.pow(3, 19)。只要判断 $ 3^19 $ 除以 `n` 余数是否为 0 就可以判断是否为 3 的幂。

```js
/**
 * 求 3 的幂
 * @param {number} n 数字
 * @return {boolean} 是否为 3 的幂
 */
function isPowerOfThree(n) {
  // 异常值处理
  if (typeof n !== 'number') return false
  return n > 0 && Math.pow(3, 19) % n == 0
}
```

#### 复杂度分析

- 时间复杂度：$ O(1) $
  该算法中，`Math.pow` 的时间复杂度为 $ O(1) $，加减乘除的时间复杂度也为 $ O(1) $，因此整体时间复杂度为 $ O(1) $。

- 空间复杂度：$ O(1) $
  该算法未使用额外的内存空间，因此空间复杂度为 $ O(1) $。


## Excel 表列序号

> 给定一个 Excel 表格中的列名称，返回其相应的列序号。

    示例

    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28
    输入: "A",
    输出: 1
    输入: "AB",
    输出: 28

详解：[LeetCode Excel 表列序号](https://leetcode-cn.com/problems/excel-sheet-column-number/solution/hua-jie-suan-fa-171-excelbiao-lie-xu-hao-by-guanpe/)

### 解法一：26进制计算法

思路：由于 Excel 表序列号只包含 [A-Z] 26个字符，且 A = 1，B = 2...我们可以将其看成是一种特殊的 26进制数，因此这道题也就变成了让你将 26进制数转 10进制数。

```js
/**
 * Excel 表列序号
 * @param {string} str 表序列号
 * @return {number} 当前为第几列
 */
function titleToNumber(str) {
  // 异常值处理
  if (typeof str !== 'string') return -1
  let sum = 0
  let i = str.length - 1
  let carry = 1
  while (i >= 0) {
    // A 的 charCode 等于 64
    sum += (str[i].charCodeAt() - 64) * carry
    carry *= 26
    i--
  }
  return sum
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法只使用了一次遍历操作，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法中临时变量的个数与循环次数无关，因此空间复杂度为 $ O(1) $。

### 解法二：利用 Hash 表快速转换

思路：直接利用 Hash 表的方式，快速获取字母对应的值，并按照位数进行累加，即 26进制转 10进制。

```js
/**
 * Excel 表列序号
 * @param {string} str 表序列号
 * @return {number} 当前为第几列
 */
function titleToNumber(str) {
  // 异常值处理
  if (typeof str !== 'string') return -1
  const arr = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
              'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
  let sum = 0
  const length = str.length
  for (let i = 0; i < length; i++) {
    sum = (arr.indexOf(str[i]) + 1) * Math.pow(26, length - 1 - i) + sum
  }
  return sum
}
```

#### 复杂度分析

- 时间复杂度：$ O(n) $
  该算法只使用了一次遍历操作，因此时间复杂度为 $ O(n) $。

- 空间复杂度：$ O(1) $
  该算法中临时变量的个数与循环次数无关，因此空间复杂度为 $ O(1) $。
