---
title: Leetcode算法大合集
date: 2021-07-05 18:08
tags: [面试,算法]
---

用于存放有价值的Leetcode算法题。

写一些有价值的算法题，虽然Leetcode上也可以写，但是还是这里方便寻找。

<!-- more -->

# 最长回文字符串

[题目传送门](https://leetcode-cn.com/problems/longest-palindromic-substring/comments/)

[答案来源传送门](https://leetcode-cn.com/problems/longest-palindromic-substring/solution/zhong-xin-kuo-san-fa-he-dong-tai-gui-hua-by-reedfa/)

## 中心扩散法

第一个想到中心扩散法，具体分析见文章。

首先肯定得遍历一遍，逐一选择“中间点”，然后往左右扩散，重点是得想到`扩散的两种情况`，

1. 由同一个字符构造成的回文。
2. 两端相同的回文。

自己盲解答时，没有考虑到第一个情况，导致解答失败。

代码如下

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function (s) {
  const len = s.length;
  let result = ''
  
  // 遍历，每一个中间点
  for (let i = 0; i < len; i++) {
    let l = i - 1;
    let r = i + 1;
    
	// 往左考虑，第一种情况
    while (l >= 0 && s[l] === s[i]) {
      l--;
    }
    
	// 往右考虑，第一种情况
    while (r < len && s[r] === s[i]) {
      r++;
    }
    
	// 考虑第二种情况
    while (l >= 0 && r < len && s[l] === s[r]) {
      r++;
      l--;
    }
    
	// 遍历每个中心点，获得此中心点的最长回文，然后获得结果
    if (result.length < r - l) {
      result = s.slice(l + 1, r)
    }
  }
  return result
};
```

## 动态规划

这是自己写的递归版动态规划，应该是中间态的动态规划。实测，耗时和耗内存都比中间扩散法大不少。

思路是：

1. 用一个二维数组dp存储状态，dp[i][j]存储s[i...j]是否是回文。
2. s[i...j]是否是回文取决于s[i+1...j-1]是否是回文。则可以得到递推公式

	f(i, j) = f(i+1, j-1) && s[i] === s[j]

3. 边界情况是，如果字符串长度是1时，则肯定是回文。字符串长度为2时，如果两个字符相同，则是回文
4. 然后考虑好边界情况，整理出一个递归公式就行。

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function (s) {
  const len = s.length;
  let result = ''
  let pre = 0;
  let after = 0;

  // 创建二维数组，用于存储。
  let dp = new Array(len)
  // 这里注意，不能 new Array(len).fill(new Array(len).fill(null))
  // fill里加new Array, 那么填充的数组是一个数组，填充的是引用。会导致改了一个，每个都会改变
  for (let i = 0; i < len; i++) {
    const newArr = new Array(len).fill(null);
    dp[i] = newArr;
  }

  for (let r = 0; r < len; r++) {
    for (let l = 0; l <= r; l++) {
      // 重要在这，递归查询
      const res = handlePalidrom(s, l, r, dp);

      if (res && r - l > after - pre) {
        after = r;
        pre = l;
      }
    }
  }

  return s.slice(pre, after + 1);
};

// [i, j]
function handlePalidrom(s, l, r, dp) {
  const len = r - l + 1;

  // 已经处理过，从缓存中获取
  if (dp[l][r] !== null) return dp[l][r];
  // 仅一个字符串的情况，肯定是回文
  if (len === 1) {
    dp[l][r] = true;
    return true;
  }
  // 两个字符串情况，需要判断两个字符串是否相等，如相等。则是回文
  if (len === 2 && s[l] === s[r]) {
    dp[l][r] = true;
    return true;
  }
  // 大于3的情况，如果两头是回文，则递归查找
  if (s[l] === s[r]) {
    const res = handlePalidrom(s, l + 1, r - 1, dp);
    if (res) {
      dp[l][r] = true;
    }
    return res;
  }
  // 其他情况，都不是回文
  dp[l][r] = false;
  return false;

}
```

// 理论上还能推出终极动态规划写法，有空再写。

## 总结

1. 回文字符串天生就是一个可以拆分成子问题的问题，动态规划是个不二之选。寻找好递推关系还是重中之重。
2. 中心扩散法貌似是最长回文问题的特殊方法，是比动态规划更优的解法。需要特殊对待

# 缺失的第一个正数

[传送门](https://leetcode-cn.com/problems/first-missing-positive/)

[好理解的最佳解-原地哈希](https://leetcode-cn.com/problems/first-missing-positive/solution/tong-pai-xu-python-dai-ma-by-liweiwei1419/)

这题目有两种答案：

## 哈希表法

`时间和空间复杂度都是O(n)`

把数组变成哈希表，然后从1开始递增。如果哈希表没有，就是答案。`哈希表`可以用对象，更佳的数据结构是Set，代码更简单（但是leetcode运行结果来看，Set更占内存、更耗时）。

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var firstMissingPositive = function (nums) {
  let set = new Set(nums);
  let result = nums.length + 1;

  for (let i = 1; i <= nums.length; i++) {
    if (!set.has(i)) {
      result = i;
      return result;
    }
  }
  return result;
};
```

## 原地哈希法

`时间复杂度O(n)，空间O(1)`

如果要求空间复杂度O(1)，则说明不需要占用额外空间，首先考虑的应该是，`要在当前数组操作`。

具体看传送门。这个思路大约是：

1. 先把负数清除，都统一转换为`null`，`排除干扰`。如果此遍历过程发现没有1。则答案肯定是1.
2. 顺序遍历，把当前遍历到的数字，把其数值的数组下标设置为负数。
	如果数值大于数组长度，则不改，否则会修改当前数组的长度。
3. 最后从1累加遍历数组，如果是负数，则对应下标的数是存在于原数组的。如果无负数，则就是当前的值。
4. 如果都是负数，结果是数组长度+1；

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var firstMissingPositive = function (nums) {
  const len = nums.length;

  let tmpArr = nums.slice();
  // 去除负数，统一改为0
  let hasOne = false;
  for (let i = 0; i < len; i++) {
    const val = tmpArr[i];
    if (val === 1) {
      hasOne = true;
    }
    if (val < 1) {
      tmpArr[i] = null	// 清空干扰
    } else {
      tmpArr[i] = val
    }
  }

  // 如果没有1。则结果是1
  if (!hasOne) return 1;

  // 对应下标的值标记为负值，不破坏原来的数字
  for (let i = 0; i < len; i++) {
    // 这里用绝对值，是因为可能被前面遍历的操作变为负值
    let val = Math.abs(tmpArr[i]);
    if (val <= len && val > 0) {
      // 把符合条件的值，对应下标的值改为负数
      tmpArr[val - 1] = -Math.abs(tmpArr[val - 1])
    }
  }

  let result = len + 1;
  // 最后走一遍，如果是负数，则原来存在于数组、非0则不存在，则是要找的结果。如果都是负数，则结果是len
  for (let i = 0; i < len; i++) {
    const val = tmpArr[i];

    // 没被标记
    if (val > 0 || val === null) {
      result = i + 1;
      return result;
    }
  }

  return result;
};
```

## 总结

1. 如果正常情况下需要O(n)空间，但是要求O(1)空间，往往是需要在`当前数组`上操作。
2. 哈希表除了`对象`以外，`Set数据结构`也许是一个更好的选择

# 对象扁平化

面试遇到的题目。leetcode暂时未找到。网上搜，有个词叫，对象扁平化。这个比对象扁平化多了个对象、数组扁平化

如下，把input转换未output的形式

```javascript
var input = {
  a: {
    a: 'a',
    b: [
      0,
      {
        a: true,
        b: 5
      }
    ]
  },
  b: [
    {
      a: 'a'
    },
    true
  ]
}

var output = {
  'a.a': 'a',
  'a.b[0]': 0,
  'a.b[1].a': true,
  'a.b[1].b': 5,
  'b[0].a': 'a',
  'b[1]': true
}
```
遇到这个问题，尝试了很多次，都不知道如何做。后来朋友实现后，发现是自己根本没有放开思维。

看代码：

```js
function faltObject(input) {
  const result = {};
  handleObject(input, result, '');
  return result;
}

function handleValue(input, res, preKey) {
  if (Array.isArray(input)) {
    handleArray(input, res, preKey)
  } else if (typeof input === 'object') {
    handleObject(input, res, preKey)
  } else {
    res[preKey] = input;
  }
}

function handleObject(input, res, preKey) {
  const entries = Object.entries(input);

  for (let [key, val] of entries) {
    const newKey = preKey ? `${preKey}.${key}` : key;
    handleValue(val, res, newKey);
  }
}

function handleArray(input, res, preKey) {
  for (let i = 0; i < input.length; i++) {
    const val = input[i];
    const newKey = preKey ? `${preKey}[${i}]` : `[${i}]`
    handleValue(val, res, newKey)
  }
}

console.log(faltObject(input));
```
自己实现时，脑子想的总是，一个方法实现，一个方法，统一输入、统一输出。最终方法里到处做判断。最终逻辑过于复杂，没有实现成功。

而正确答案，从头到尾操作的只是一个全局的对象result，只是递归的堆叠key，直到递归到非对象和数组时，给全局result对象赋值即可。这么一想就简单很多。

## 总结

1. 写算法就不用要求写pure function。可以的话用全局变量可以简化一些逻辑。
2. `Object.entries()` +` for(let [key, value] of entries)`是个更好的遍历对象的方法，可以替代`Object.keys().forEach()`
3. 不用考虑得太复杂，一开始就封装了个isObject。而实际情况，直接`typeof xx === 'object'`就差不多了。


# 寻找两个字符串中重复的部分



