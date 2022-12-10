---
title: "零錢問題"
date: 2022-02-19T13:33:31+08:00
draft: false
toc: true
images:
tags: 
  - 程式
---

## 簡介

零錢問題是動態規劃(dynamic programming, 簡稱dp)常見的問題之一，題目敘述可能是問有幾 n 錢幣的情況下，付 m 元有幾種組合，或者付 m 元所需要的最少硬幣數。

## 解題概念

首先會需要紀錄「有幾種錢幣可用」以及「1 ~ n 元個別所需的組合/最少硬幣數紀錄」。

```cpp
int prices[5]={1,5,10,25,50};
int c[5000]={1};
```

c[0]之所以為 1，是因為若 0 元的組合只會有一種情況：不用任何一種硬幣。

### 求組合數

```cpp
for (int i = 0; i < n; i++)
{
    for (int j = prices[i]; j <= m; j++)
        c[j] += c[j - prices[i]];
}
```

第一個迴圈，是把所有種類的硬幣一個個放進去。第二個迴圈，是把從現在放入的硬幣幣值 prices[i] ~ m 都看過一遍。

檢查的方法，就是 `c[j] += c[j - prices[i]];`。可以理解為當我多放了 prices[i] 這種硬幣時，除了我 c[j] 目前有的方法數外，還能額外得到 c[j-prices[i]] 的方法數，因為只要多放 prices[i] 就好了。

例：
* [d253: 00674 - Coin Change](https://zerojudge.tw/ShowProblem?problemid=d253)
* [d119: 有獎徵答：換零錢](https://zerojudge.tw/ShowProblem?problemid=d119)

### 求最少硬幣數

```cpp
for (int i = 0; i < n; i++)
{
    for (int j = price[i]; j <= c; j++)
    {
        money[j] = min(money[j], money[j - price[i]] + 1);
    }
}
```

看是哪一種情況比較少，再更新紀錄即可。

例：
* [d904: 換零錢](https://zerojudge.tw/ShowProblem?problemid=d904)
* [c317: 硬幣問題！前傳](https://zerojudge.tw/ShowProblem?problemid=c317)

## 變化題

例：
* [d289: 多元一次方程式](https://zerojudge.tw/ShowProblem?problemid=d289)

與其說這題是零錢問題的變化題，不如說零錢問題的本質就是解特定的多元一次方程式。未知數的係數是硬幣種類，未知數本身是組成的方式，等式右邊的常數是要解的目標額度。

* [b232: TOI2009 第四題：分房子](https://zerojudge.tw/ShowProblem?problemid=b232)

