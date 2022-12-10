---
title: "機器學習基石 作業一"
date: 2021-09-19T13:25:40+08:00
draft: false
toc: True
images:
tags: 
  - 機器學習
  - coursera
---

## Questions 15~17
根據講義的演算法進行實作:
![](https://i.imgur.com/FgwvbSQ.png)

### 資料前處理
```python
import numpy as np

data = np.array([i.strip().split() for i in open("./data.dat").readlines()])
X_train = data[:, :4]
Y_train = data[:, -1]

xzero = np.ones((X_train.shape[0], 1))
X_train = np.concatenate((xzero, X_train), axis=1).astype('float')
Y_train = Y_train.astype('float')
```
### 會用到的函式
```python
def sign(x):
    if x <=0 :
        return -1
    else:
        return 1
    
def _shuffle(X, Y):
    index = np.arange(len(X))
    np.random.shuffle(index)
    return X[index], Y[index]

def PLA(X_train, Y_train, shuffle=False, eta=1):
    update_time = 0
    w = np.zeros((X_train.shape[1], 1))

    if shuffle == True:
        X_train, Y_train = _shuffle(X_train, Y_train)

    while True:
        isFinish = True

        for i in range(X_train.shape[0]):

            if sign(np.dot(X_train[i], w)) != Y_train[i]:
                w = w + eta*(Y_train[i] * X_train[i]).reshape(-1, 1)
                isFinish = False
                update_time += 1

        if isFinish == True:
            break
    return update_time
```

### 執行
```python
# 第15題
PLA(X_train, Y_train)

# 第16題
total_update = 0
for i in range(2000):
    total_update += PLA(X_train, Y_train, shuffle=True)
print(total_update / 2000)

# 第17題
total_update = 0
for i in range(2000):
    total_update += PLA(X_train, Y_train, shuffle=True, eta=0.5)
print(total_update / 2000)
```

## Questions 18~20
修改PLA的部分內容即可。
![](https://i.imgur.com/7HBO3Jr.png)

### 資料前處理、函式定義
```python
import numpy as np

train_data = np.array([i.strip().split() for i in open("./train.dat").readlines()])
test_data = np.array([i.strip().split() for i in open("./test.dat").readlines()])

X_train = train_data[:, :4]
Y_train = train_data[:, -1]
X_test = test_data[:, :4]
Y_test = test_data[:, -1]

xzero = np.ones((X_train.shape[0], 1))
X_train = np.concatenate((xzero, X_train), axis=1).astype('float')
Y_train = Y_train.astype('float')

xzero = np.ones((X_test.shape[0], 1))
X_test = np.concatenate((xzero, X_test), axis=1).astype('float')
Y_test = Y_test.astype('float')

def sign(x):
    if x <=0 :
        return -1
    else:
        return 1
    
def _shuffle(X, Y):
    index = np.arange(len(X))
    np.random.shuffle(index)
    return X[index], Y[index]
```

### pocket
```python
def pocket(X_train, Y_train, shuffle=False, eta=1):
    update_time = 0
    min_error = X_train.shape[0]
    w = np.zeros((X_train.shape[1], 1))
    w_pocket = w.copy()

    if shuffle == True:
        X_train, Y_train = _shuffle(X_train, Y_train)

    for i in range(X_train.shape[0]):
        error = 0
            
        if sign(np.dot(X_train[i], w)) != Y_train[i]:
            w = w + eta*(Y_train[i] * X_train[i]).reshape(-1, 1)
            isFinish = False
            update_time += 1
                
            for j in range(X_train.shape[0]):
                if sign(np.dot(X_train[j], w)) != Y_train[j]:
                     error += 1
            if error < min_error:
                min_error = error
                w_pocket = w.copy()
        
        if update_time == 50:
            break
    return w_pocket

def get_error(X, Y, w):
    error = 0
    for i in range(X.shape[0]):
        if sign(np.dot(X[i], w)) != Y[i]:
            error += 1
    return error/X.shape[0]                            
```

### 執行
根據題目要求修改pocket函式部分內容，重跑執行區塊即可。

```python
total_error = 0
for i in range(2000):
    w_pocket = pocket(X_train, Y_train, shuffle=True)
    total_error += get_error(X_test, Y_test, w_pocket)
    
    if i%100 == 0:
        print(i)
        
total_error/2000
```