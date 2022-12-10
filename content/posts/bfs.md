---
title: "BFS(Breadth First Search) 筆記"
date: 2022-02-08T11:17:22+08:00
draft: false
toc: true
images:
tags: 
  - 程式
---

## 使用時機
需要找最短距離(shortest path)，或者遍歷圖(traversal graph)的時候。

## 演算法
給定一個起始的節點，並逐一尋找相鄰解點直到找到特定的節點或遍歷完整張圖。
![](https://i.imgur.com/HBxapW9.png)

## 題目類型
1) 給定 edges ，問能否從 A 點走到 B 點

* [a290: 新手訓練系列 ~ 圖論](https://zerojudge.tw/ShowProblem?problemid=a290)

2) 給定 A、B 兩點，問最短路徑 or 走迷宮

* [c124: 00532 - Dungeon Master](https://zerojudge.tw/ShowProblem?problemid=c124)
* [d453: 三、最短距離](https://zerojudge.tw/ShowProblem?problemid=d453)
* [d406: 倒水時間](https://zerojudge.tw/ShowProblem?problemid=d406)

3) 尋找、探索特定區域

* [c129: 00572 - Oil Deposits](https://zerojudge.tw/ShowProblem?problemid=c129)
* [d365: 10336 - Rank the Languages](https://zerojudge.tw/ShowProblem?problemid=d365)
* [e550: 00722 - Lakes](https://zerojudge.tw/ShowProblem?problemid=e550)
* [f435: 10267-Graphical Editor](https://zerojudge.tw/ShowProblem?problemid=f435)

## 解法筆記
### 類型一
假設有 n 個節點，則用 n * n 的二維陣列來記錄點與點間是否連通(需注意 edge 單向或雙向)。

用一維陣列紀錄某點是否走過，避免回頭的情況發生。

```cpp
int graph[100][100]={0};
bool isVidited[100]={false;}
```

### 類型二
假設一個點可以拜訪地圖上下左右四個位置，與其寫 4 個 if 有更簡潔的寫法

```cpp
int dx[4] = { 1,0,0,-1 }, dy[4] = { 0,1,-1,0 }; 
for(int i = 0; i < 4; i++)
{
    if (p.x >= 0 && p.x < n && p.y >= 0 && p.y < m && map[p.x][p.y] != 0 && !isVisited[p.x][p.y])
        // do somthing
}
```

事先列舉好需要的位移量，透過 for loop 逐一拜訪，若該點合法則進行相關操作。

### 類型三
這類題目有時會給特定的某點，看該點所在的範圍面積；也可能沒有特定的起始點，需要檢查完整張圖。

前者需要每從 queue 拿出一個點(也就是拜訪了某點)後將 counter 加一來計算。後者可以寫一個bfs函式，透過 traverse 整張圖的方式，探索範圍大小等等。
```cpp
for (int i = 0; i < m; i++)
{
	for (int j = 0; j < n; j++)
	{
		if (map[i][j] == '@' && !isVisited[i][j])
			bfs(i, j, '@'), ans++;
	}
}
```
## 程式範例
以「[d406: 倒水時間](https://zerojudge.tw/ShowProblem?problemid=d406)」為例，因應題目描述不同而略有變化，但其核心大同小異。
```cpp
#include <iostream>
#include <queue>
using namespace std;

int map[101][101] = { 0 };
struct Point
{
	int x, y, step = -1;
};

int main()
{
	int s, n, m, counter = 1;
	while (cin >> s)
	{
		cin >> n >> m;
		bool isVisited[101][101] = { false };
		Point start, now;
		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < m; j++)
			{
				cin >> map[i][j];
				if (i == 0 && map[i][j] == 1)
					start.x = i, start.y = j;
			}
		}
		queue<Point> q;
		isVisited[start.x][start.y] = true;
		start.step = 1;
		q.push(start);
		int dx[4] = { 1,0,0,-1 }, dy[4] = { 0,1,-1,0 }; // 最後一個是往上
		while (!q.empty())
		{
			now = q.front(); q.pop();
			for (int i = 0; i < 4; i++)
			{
				if (i == 3 && s == 2) break;
				Point p;
				p.x = now.x + dx[i]; p.y = now.y + dy[i];
				if (p.x >= 0 && p.x < n && p.y >= 0 && p.y < m && map[p.x][p.y] != 0 && !isVisited[p.x][p.y])
				{
					isVisited[p.x][p.y] = true;
					p.step = now.step + 1;
					map[p.x][p.y] = p.step;
					q.push(p);
				}
				
			}
			isVisited[now.x][now.y] = true;
		}
		cout << "Case " << counter++ << ":" << endl;
		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < m; j++)
			{
				if (map[i][j] != 0 && !isVisited[i][j]) cout << 0;
				else cout << map[i][j];
				if (j != m - 1) cout << " ";
				else cout << endl;
			}
		}
	}
	return 0;
}
```