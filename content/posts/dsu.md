---
title: "Disjoint Set Union(DSU) 筆記"
date: 2022-02-28T22:10:37+08:00
draft: false
toc: true
images:
tags: 
  - 程式
---

## 簡介

我們可能會有好多個元素 (element) 分別屬於不同的群體 (set)，透過了解元素 A 跟元素 B、元素 C 跟元素 D 誰跟誰有關，隨時依照需求更新並建立 element 間彼此的關係。在這類的問題中，我們關心的問題主要有二：

* A 跟 B 屬於同一個群體嗎？
* X 類別的數量有多少？

最直接的方式，是利用一維陣列來記錄彼此間的關係。index 表示哪個 element，裡面存的數值表示它指向哪個 element，也就是跟誰有關。這樣的關係就相當於建立了一條長鏈，該鍊的盡頭就是這一群體的「老大」。當我們想知道元素們是否屬於同的群體，只要看「老大」一不一樣就好了。

## 程式碼片段

* make_set()
```cpp
void make_set()
{
    for(int i = 0; i < n; i++)
        dsu[i] = i;
}
```

`make_set()` 的目的是初始化。在最一開始，每一個 element 自己都是「老大」。

* find_set()

`find_set()` 要做的，就是找到 element v 的「老大」是誰。

```cpp
int find_set(int v)
{
    if(dsu[v] == v) return v;
    return dsu[v] = find_set(dsu[v]);
}
```
第一行就是搜尋走到最末端，找到老大的時候。第二行的用意是「透過遞迴找到 root 並且回傳」，但當建立起 set 中元素間的關係時，可能會是長長一串，這大幅降低了搜尋的效率，為此我們可以做 "path compression"。這個方法讓從最開始找的元素到 root 這條長鍊被壓縮，每個元素直接指向 root。

另外也有 union by size, union by rank 等優化方法。

* union()
```cpp
void union(int u, int v)
{
    u = find_set(u), v = find_set(v);
    if(u != v)
    {
        dsu[u] = v;
    }
}
```
`union()` 是把兩個 set 合併在一起，至於怎麼合併若無特殊要求則隨意。

值得注意的是，雖然我們輸入的是 u 跟 v 兩個元素，但實際上只關心他們所屬群的「老大」。不管是合併，還是要更新群體的數量，都是從「老大」來弄。

## 題目類型

基本題
* [a445: 新手訓練系列- 我的朋友很少](https://zerojudge.tw/ShowProblem?problemid=a445)
* [f677: FJCU_109_Winter_Day3_Lab1 並查集練習](https://zerojudge.tw/ShowProblem?problemid=f677)

變形題
* [f260: 愛八卦的同學](https://zerojudge.tw/ShowProblem?problemid=f260)
* [d831: 畢業旅行](https://zerojudge.tw/ShowProblem?problemid=d831)

