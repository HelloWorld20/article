---
title: leetcode
date: 2021-04-22 14:03:24
tags: [总结,javascript,leetcode]
---


```javascript
var longestPalindrome = (raw) => {

  if (raw.length === 1) return 1;

  let result = 0;
  for (let i = 0; i < raw.length; i++) {
    for (let j = i + 1; j <= raw.length; j++) {
      let tempStr = raw.slice(i, j);
      if (isPalindrome(raw.slice(i, j))) {
        const palindromeLen = tempStr.length;
        if (palindromeLen > result) {
          result = palindromeLen
        }
      }
    }
  }

  return result;
}

function isPalindrome(str) {
  let headIdx = 0;
  let tailIdx = str.length - 1;
  let result = true;
  while (headIdx < tailIdx && result) {
    result = str[headIdx] === str[tailIdx]

    headIdx++;
    tailIdx--;
  }

  return result;
}
```