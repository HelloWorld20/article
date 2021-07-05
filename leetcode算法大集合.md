---
title: Leetcode算法大合集
date: 2021-07-05 18:08
tags: [面试,算法]
---

用于存放有价值的Leetcode算法题。

写一些有价值的算法题，虽然Leetcode上也可以写，但是就不写那了，这里方便寻找

<!-- more -->

# 最长回文字符串

[题目传送门](https://leetcode-cn.com/problems/longest-palindromic-substring/comments/)

[答案来源传送门](https://leetcode-cn.com/problems/longest-palindromic-substring/solution/zhong-xin-kuo-san-fa-he-dong-tai-gui-hua-by-reedfa/)

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

