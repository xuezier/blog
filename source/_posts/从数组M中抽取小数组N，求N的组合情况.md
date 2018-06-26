---
title: 从数组M中抽取小数组N，求N的组合情况
date: 2017-04-06 13:44:20
tags:
  - javascript
  - Array
---
算法题：从数组M中抽取小数组N，求N的组合情况，N的内容用下标表示。比如M：[0,1,2,3]，取N的长度为3，结果的N集合为：[0,1,2],[0,1,3],[0,2,3],[1,2,3]。只考虑组合，不考虑排序，例如 [0,1,2], [0,2,1] 只算一个。

## 确定关系
M, N 是不确定的两个数组，N 是 M 满足情况的子集，M.length = m, N.length = n, n < m.

以 m=10, n=4为例。
```js
const m = 10, n = 4;
```

## 创建数组 M
```js
const M = Array.from({length: m}).map((it,i)=>{return i;});
```

## 求取数组 N 的可能性
我们通过确定每一位的情况，逐位向后填充，直至取到满足 N 的数组.

### 先确定第一位的情况。
```js
let N = [];
for(let i = 0; i < m - n + 1; i++){
  /**
  * N 的每种情况都按顺序填充
  * 当 m = 10，n = 4，N[0] 的取值范围应该是 [0~6] 即 m - n
  */
  N[0] = i;
}
```

### 通过迭代向后填充
确定了第一位的情况后，我们可以以此类推，通过迭代，将下一位的每种情况进行填充

将方法封装
```js
const N_1 = N - 1;
function getNextIndex(preN = [], index = 0, MIndex = 0){
  for(let i = 0; i < m - MIndex - n + index + 1; i++){
    let N = [];  // 创建新的数组备份，将上一次求到 N 的内容进行拷贝
    for(let t = 0; t < index; t++){
      N[t] = preN[t];
    }
    N[index] = M[MIndex + i];
    index < N_1 ? getNextIndex(N, index + 1, i + MIndex +1) : null;
  }
}
```

这样就求出了所有按顺序排列的 N 的可能组合。

### 综合
```js
const m = 10, n = 4;
const M = Array.from({length: m}).map(function(it, i)=>{return i;});
function arrayFilter(){
  let N_RESULTS = [], RESULTS_MARK = 0;
  const N_1 = n - 1;
  getNextIndex();
  function getNextIndex(preN = [], index = 0, MIndex = 0){
    for(let i = 0; i < m - MIndex - n + index + 1; i++){
      let N = [];  // 创建新的数组备份，将上一次求到 N 的内容进行拷贝
      for(let t = 0; t < index; t++){
        N[t] = preN[t];
      }
      N[index] = M[MIndex + i];
      index < N_1 ? getNextIndex(N, index + 1, i + MIndex +1) : (N_RESULTS[RESULTS_MARK++] = N);
    }
  }
}
```

## Notes
这是整理过后的代码，去掉了很多数组操作，相比旧的代码，性能提升明显

旧的算法
```js
const m = 10, n = 4;
const M = Array.from({length: m}).map(function(it, i)=>{return i;});
function oldarrayFilter() {
  let N_RESULTS = [];
  getNextIndex();
  function getNextIndex(preN = [], index = 0, arrayIndex = 0) {
    let N = Array.from(preN);
    let nextArray = M.slice(arrayIndex);
    for (let i = 0; i < nextArray.length - n + index + 1; i++) {
      N[index] = nextArray[i];
      (N.length < n) ? getNextIndex(N, index + 1, nextArray[i] + 1): N_RESULTS.push(N);
    }
  }
}
```

通过 benchmark 这个库来检测新旧算法的效率
```bash
> node arrayFilter.js
new x 48,368 ops/sec ±1.41% (82 runs sampled)
old x 1,322 ops/sec ±1.83% (77 runs sampled)
Fastest is new
```

主要差异是旧的算法用了较多的数组操作，虽然理解上更方便，写法更直接，却造成了较多的性能开销.