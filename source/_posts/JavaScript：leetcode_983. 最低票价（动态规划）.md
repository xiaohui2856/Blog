---
title: JavaScript：leetcode_983. 最低票价（动态规划）
date: 2020-05-06 17:59:13
categories: ['Algorithm','每日一题']
tags: Algorithm
keywords: leetcode, 最低票价, 动态规划
---

### 题目说明
```

在一个火车旅行很受欢迎的国度，你提前一年计划了一些火车旅行。在接下来的一年里，你要旅行的日子将以一个名为 days 的数组给出。每一项是一个从 1 到 365 的整数。

火车票有三种不同的销售方式：

一张为期一天的通行证售价为 costs[0] 美元；
一张为期七天的通行证售价为 costs[1] 美元；
一张为期三十天的通行证售价为 costs[2] 美元。
通行证允许数天无限制的旅行。 例如，如果我们在第 2 天获得一张为期 7 天的通行证，那么我们可以连着旅行 7 天：第 2 天、第 3 天、第 4 天、第 5 天、第 6 天、第 7 天和第 8 天。

返回你想要完成在给定的列表 days 中列出的每一天的旅行所需要的最低消费。

 

示例 1：

输入：days = [1,4,6,7,8,20], costs = [2,7,15]
输出：11
解释： 
例如，这里有一种购买通行证的方法，可以让你完成你的旅行计划：
在第 1 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 1 天生效。
在第 3 天，你花了 costs[1] = $7 买了一张为期 7 天的通行证，它将在第 3, 4, ..., 9 天生效。
在第 20 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 20 天生效。
你总共花了 $11，并完成了你计划的每一天旅行。
示例 2：

输入：days = [1,2,3,4,5,6,7,8,9,10,30,31], costs = [2,7,15]
输出：17
解释：
例如，这里有一种购买通行证的方法，可以让你完成你的旅行计划： 
在第 1 天，你花了 costs[2] = $15 买了一张为期 30 天的通行证，它将在第 1, 2, ..., 30 天生效。
在第 31 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 31 天生效。 
你总共花了 $17，并完成了你计划的每一天旅行。
 

提示：

1 <= days.length <= 365
1 <= days[i] <= 365
days 按顺序严格递增
costs.length == 3
1 <= costs[i] <= 1000

```
<!-- more -->

### 解题思路一

动态规划的问题。

好多题解是倒着来算的，对于书写是更简洁了，但是对于思维，太别扭了。。我脑子实在转不过来。

所以我就换成正序规划，这样好理解一点。

此题有点类似于背包算法。但是又不太一样。



1. 打点 days[0] 到 days[days.length - 1]的每一天。 所以初始化 dp = new Array(days.length).fill(0),代表的是`一年的第一天`到`计划旅行的最后一天`
2. days[0]的值为计划开始旅行的第一天的日期，所以当开始第一天时，当然是2块钱最划算，所以dp[days[0]] = 2
3. 第三步就比较核心了!!!!
    1. 当我们算到第二天的时候，是不是要比较买什么票合适呢？
    2. 买如果只买一天的票，那就是今天之前所有的money加上今天的2块钱，`dp[i-1] + costs[0]`
    3. 如果买7天的票，那就是今天往前划拉7天的money加上今天的7块钱，`dp[i-7] + costs[1]`
    4. 同理买30天的票，划拉30天`dp[i-30] + costs[2]`
    5. 好嘞，我们又三种选择了，根据题意和常识，我们选择最`便宜的。`用`Math.min(dp[i-1] + costs[0],dp[i-7] + costs[1],dp[i-30] + costs[2])`



### 代码实现一
```javascript
/**
 * @param {number[]} days
 * @param {number[]} costs
 * @return {number}
 */
var mincostTickets = function(days, costs) {
    let dp = new Array(days[days.length-1]).fill(0);
    for (let i = days[0], k = 0; i <= days[days.length-1]; i++) {
        if (i === days[k]) { //确定今天是不是旅行日
            dp[i] = Math.min(dp[(i - 1)>=0?(i - 1):0] + costs[0],
                             dp[(i - 7)>=0?(i - 7):0] + costs[1],
                             dp[(i - 30)>=0?(i - 30):0] + costs[2]) //如果是，就得用前面花的钱加上今天花的钱。
            //今天之前的钱数都是确定的且最小的。
            k++
        } else {
            dp[i] = dp[i-1] //如果今天不旅行，那肯定不花钱，跟前一天的钱一样。
        }
    }
    return dp[dp.length-1]
};
```