---
layout: post
title: 排列组合算法[Python]
categories: [编程学习]
tags: [算法, 排列组合]
date: 2017-03-11
description: 最新学到的排列组合算法
keywords: 排列组合, Python, 算法, powerset, Algorithms
---

# 两种排列组合算法

* 一个是在Edx课上看到的，一个是Python的源码

## 通过二进制中“1”所在位置的可能性来确定数组中的索引位置，进而求得所有排列组合

* 首先确定组合的数量是2的N次方个，然后循环2^N， 每个数字即代表一种可能。
  * 比如数组长度是5的情况下，我们一共有2^5种可能，第 1 种可能 对应的二进制为 `0 0 0 0 1`，第 5 种 对应的是 `0 1 0 0 1`.
* 其次既是如何把对应位置的二进制转化成对应数组的索引位置，比如上例中 第 5 种可能 即为 5 转换的二进制：`0 1 0 0 1`，那么它对应的数据应该是数组种的 第二位和最后一位
  * 算法中，通过 `(i >> j) % 2 == 1` 来确定 当前二进制位是否为 1，在此就不赘述了。

```python
def powerSet(items):
    N = len(items)
    # enumerate the 2**N possible combinations
    for i in range(2**N):
        combo = []
        for j in range(N):
            # test bit jth of integer i
            # test bit jth of integer i
            # >>j. move the bit we want to check to the end
            # %2. remove all the other bits execpt the last one
            # check the one we kept if it is 1 not 0,
            # which means we want to keep the item which on the position
            # example:  0 1 1 0 1
            # we want to check the third "1"
            # first move the second bit to the end(>>j), will be "0 0 0 1 1"
            # then remove all the other bits(%2), we got "0 0 0 0 1"
            # compare it with 1, which is true, 
            # so we take the item with the position, which will be item[2]
            if (i >> j) % 2 == 1:
                combo.append(items[j])
        yield combo
```

## 使用python官方文档提供的`combinations`

* 为了更好的理解这个算法，我把它单独拿了出来，并没有导入
* 同上一个算法，这个也是遍历找得所有的索引，然后取出数组中对应的数据
* 之前很努力的写了英文 不想翻译了，勉强看啦

```python
from itertools import chain

def powerset_generator(sets):
    for subset in chain.from_iterable(combinations(sets, r) for r in range(len(sets)+1)):
        yield subset
        
# the logic of this function is 
#   set a new array with length r
#   loop the last element's index from i to i+n-r(n is the length of pool, r is the length of subsequence). 
#   when hit the maximum which should be n-1, increase the last-1 element's index.
#   loop until the first element's index hit the maximum, 
#   then increase the previous index, and set the last index to previous index + 1, 
#   then back to the loop until all of the indices hit the maximum
# For example: iterable = [1,2,3,4,5], r = 3
#   (1, 2, 3)
#   (1, 2, 4)
#   (1, 2, 5) <-- the last index hit the maximum
#   (1, 3, 4) <-- increase the previous index, and set every one after to previous index + 1,
#   (1, 3, 5) 
#   (1, 4, 5) <-- the (last-1) index hit the maximum
#   (2, 3, 4)
#   (2, 3, 5)
#   (2, 4, 5)
#   (3, 4, 5) <-- the (last-2) index hit the maximum
def combinations(iterable, r):
    
    pool = tuple(iterable)
    
    n = len(pool)
    
    if r > n:
        return
    indices = list(range(r))
 
    # In the "while" circle, we will start to change the indices by adding 1 consistently.
    # So yield the first permutation before the while start.
    yield tuple(pool[x] for x in indices)
    
    while True:
    
        # This 'for' loop is checking whether the index has hit the maximum from the last one to the first one.
        # if it indices[i] >= its maximum, 
        #   set i = i-1, check the previous one
        # if all of the indices has hit the maximum, 
        #   stop the `while` loop
        for i in reversed(range(r)):
            
            # let's take an example to explain why using i + n - r
            # pool indices: [0,1,2,3,4]
            # subsequence indices: [0,1,2]
            # so
            #   indices[2] can be one of [2,3,4],
            #   indices[1] can be one of [1,2,3],
            #   indices[0] can be one of [0,1,2],
            # and the gap of every index is n-r, like here is 5-3=2
            # then
            #   indices[2] < 2+2 = i+2 = i+n-r,
            #   indices[1] < 1+2 = i+2 = i+n-r,
            #   indices[0] < 0+2 = i+2 = i+n-r,
            if indices[i] < i + n - r:
                break
        else:
            # loop finished, return
            return
        
        # Add one for current indices[i], 
        # (we already yield the first permutation before the loop)
        indices[i] += 1
        # this for loop increases every indices which is after indices[i].
        # cause, current index has increased, and we need to confirm every one behind is initialized again.
        # For example: current we got i = 2, indices[i]+1 will be 3, 
        # so the next loop should start with [1, 3, 4], not [1, 3, 3]
        for j in range(i+1, r):
            indices[j] = indices[j-1] + 1
            
        yield tuple(pool[x] for x in indices)  
```

## 源地址

* [https://cs.ericyy.me/computational-thinking/lecture-2-powerset.html](https://cs.ericyy.me/computational-thinking/lecture-2-powerset.html)

## 参考

* [按位操作符（Bitwise operators）from MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)
