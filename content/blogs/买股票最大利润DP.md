---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "买股票最大利润DP"
date: 2020-06-07T13:38:18+08:00
lastmod: 2020-06-07T13:38:18+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: ["OJ","DP"]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

## 彩票类题目描述
- 给定一个数组prices,第i项表示第i天买入或卖出彩票的价格，买入之前必须将当前手里的彩票卖出，一个买入卖出组合被称作一次交易，一共可以进行K次交易，最大利润是多少？

## 状态表示和转移
- 每天有买入，卖出，不操作三种情况，并且其中买入次数收到K的限制，所以需要确定下来天数i,当前剩余的买入次数k，和当日操作结束后是否拥有股票来确定唯一一个状态。
```c++
/* 状态表示 */
// 表示前i天共拥有k次交易次数且第i天操作结束后没有拥有股票时的最大利润。
dp[i][k][0];  
// 表示前i天共拥有k次交易次数且第i天操作结束后拥有股票时的最大利润。
dp[i][k][1];
```
- 而对于第i天，其操作结束后是否拥有股票和第i-1天有关。
  
```c++
/* 状态转移 */
/* 
    第i天操作后不拥有股票：
        第i-1天操作后不拥有股票，且第i天没有操作
        第i-1天拥有股票且第i天卖出
*/
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]);
/* 
    第i天操作后拥有股票：
        第i-1天操作后拥有股票，且第i天没有操作
        第i-1天不拥有股票且第i天买入
*/
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i]);
//此处用k表示剩余买入的次数，表示剩余卖出的次数结果一样。
```

- 考虑初始化状态，第0天的时候，如果操作后不拥有股票，说明没有操作，利润位0，反之如果第0天时还剩余交易次数，且操作后拥有股票则说明第0天买入，利润为**0 - prices[0]**。

```c++
for(int i = 0; i <= k; ++i){
    dp[0][i][0] = 0;
    dp[0][i][1] = 0 - prices[0];
}
dp[0][0][1] = INT_MIN; //(用INT_MIN表示不可能)
```
