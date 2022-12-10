---
title: "質數"
date: 2022-01-26T11:22:03+08:00
draft: false
toc: true
images:
tags: 
  - 程式
---

## 前言

在解質數相關問題時，往往題目有時間限制，如果每個數字 n 都要重新把 2 ~ sqrt(n) 的數字都除除看來判斷，很有可能就 TLE。

為此，我們需要事先透過「質數篩法」來建表。

## 方法

質數篩法的概念不難。先假設宣告一個初始值為真的陣列，其 index 對應到的值就表示該數是否為質數，也應此需要將 index 為 0、1 的值改為 false。

於是目前陣列第一個為 true 的就是數字 2，這代表它是質數，也代表它的倍數必定不為質數。所以接下來要做的，就是把所有 2 的倍數一一設為 false。

同樣的方法再來一遍，會把所有數字 3 的倍數設為 false。接下來是 5、7、11......以此類推。

## 實作
```cpp
#include <iostream>
using namespace std;

#define size 5000

int main()
{
	bool prime[size] = { 0 }; // 先假設找size內的質數
	for (int i = 2; i < size; i++)
		prime[i] = true; // 方便後續操作的初始化

	for (int i = 2; i < size; i++) // 只需要檢查到長度開根號
	{
		if (prime[i]) // 如果該數還沒被改動過，就是質數
		{
			for (int j = 2 * i; j < size; j += i) // 把後面它的倍數通通設false
				prime[j] = false;
		}
	}
	
	for (int i = 1; i < size; i++) // 印出來檢查
	{
		if (prime[i])
			cout << i << endl;
	}
	return 0;
}

// 示意圖
// 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
// F T T T T T T T T  T  T  T  T  T  T <- 初始值
// F T T F T F T F T  T  T  F  T  F  T <- 選2
// F T T F T F T F F  F  T  F  T  F  F <- 最後結果
```

## 題目練習
### ZeroJudge a007: 判斷質數
這題需要注意的是，陣列 size 不能照題目敘述開 2147483647 的大小，不然會爆掉。

既然如此，就退一步用 sqrt(2147483647) 也就是建表建到 46341 吧！超出這個大小，回頭檢查能不能被目前已知質數給除盡，小於這個大小可以直接查表。

```cpp
#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

#define Size 46342
bool prime[Size] = { 0 }; // 先假設找size內的質數
vector<int> isPrime;

void sieve()
{
	for (int i = 2; i < Size; i++)
		prime[i] = true; // 方便後續操作的初始化

	for (int i = 2; i < Size; i++) // 只需要檢查到長度開根號
	{
		if (prime[i]) // 如果該數還沒被改動過，就是質數
		{
			isPrime.push_back(i);
			for (int j = 2 * i; j < Size; j += i) // 把後面它的倍數通通設false
				prime[j] = false;
		}
	}
}

int main()
{
	
	sieve();

	int n;
	while (cin >> n)
	{
		if (n > Size)
		{
			bool notPrime = false;
			for (int i = 0; i < isPrime.size(); i++)
			{
				if (n % isPrime[i] == 0)
				{
					notPrime = true;
					cout << "非質數" << endl;
					break;
				}
			}
			if (!notPrime)
				cout << "質數" << endl;
		}
		else
		{
			if (prime[n])
				cout << "質數" << endl;
			else
				cout << "非質數" << endl;
		}
	}
	return 0;
}
```