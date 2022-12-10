---
title: "CDMC2021 task3"
date: 2021-10-10T14:18:02+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - note
---

## 目標

因為跟syscall很相似，我就用了相近的方法處理，不過實作細節略有不同。

整體而言，都是找出出現過的string類型，依照出現次數進行預測。

## 程式

```python
import numpy as np
import pandas as pd

data = [i.strip().split() for i in open("./CDMC2021_IoTMalware_Train.data").readlines()]

word_list = []
for row in data:
  for line in row:
    words = line.split(';')
    for word in words:
      word_list.append(word)
#word_list
```
word_list是紀錄所有出現過的類型。因為原始資料會以分號隔開不同node，但我的作法不考慮node的影響，所以會直接切開。

```python
from collections import Counter
words_to_count = (word for word in word_list)
c = Counter(words_to_count)

max_number = 25000

top_word = c.most_common(max_number)
top_wordlist = []
for i, word in enumerate(top_word):
  top_wordlist.append(word[0])
```
這裡利用Counter，算出出現過的類型跟次數，並且以dict型態回傳。

因為取全部類型在colab運行時，記憶體會炸掉，所以max_number只取最常出現的25000種。

top_word得到前25000種的dict，接著把key單獨取出來。

```python
word_listoflist = []
for row in data:
  l = []
  for line in row:
    words = line.split(';')
    for word in words:
      l.append(word)
  word_listoflist.append(l)
```
前面是把所有資料都一股腦塞進一個list，這邊則是每筆資料一個list，統整到一個list中，方便後續處理。

```python
X = np.zeros((len(data), max_number))
for i in range(len(data)):
  words_to_count = (word for word in word_listoflist[i])
  row_count = Counter(words_to_count)

  for j in range(max_number):
    if top_wordlist[j] in row_count:
      X[i][j] = row_count[top_wordlist[j]]
```
針對選出來的25000個種類，看每筆data的出現次數。

再來就差不多能進行預測了。

```python
Y = np.array([i.strip().split() for i in open("./CDMC2021_IoTMalware_Train.label").readlines()]).astype('int')
Y = Y.reshape(-1, )

from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X)
X = scaler.transform(X)

# split training set and testing set
from sklearn.model_selection import train_test_split
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.25, random_state=66)

from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(random_state=66)
clf.fit(X_train, Y_train)
print('train acc:', clf.score(X_train, Y_train))
print('test acc:', clf.score(X_test, Y_test))

from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(random_state=66)
rf.fit(X_train, Y_train)
print('train acc:', rf.score(X_train, Y_train))
print('test acc:', rf.score(X_test, Y_test))
```