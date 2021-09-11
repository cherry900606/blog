---
title: "李宏毅機器學習2020_hw2"
date: 2021-08-16T14:01:22+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - note
---

## 範例code

### preparing data

```python
X_train_fpath = './data/X_train'
Y_train_fpath = './data/Y_train'
X_test_fpath = './data/X_test'
output_fpath = './output_{}.csv'

# Parse csv files to numpy array
with open(X_train_fpath) as f:
    next(f)
    X_train = np.array([line.strip('\n').split(',')[1:] for line in f], dtype = float)
with open(Y_train_fpath) as f:
    next(f)
    Y_train = np.array([line.strip('\n').split(',')[1] for line in f], dtype = float)
with open(X_test_fpath) as f:
    next(f)
    X_test = np.array([line.strip('\n').split(',')[1:] for line in f], dtype = float)
```

> strip(): 除去開頭跟尾端的特定字元。

> split(): 用特定字元切割字串。

> [line.strip('\n').split(',')[1:] for line in f]: list comprehension
>
> 遍歷file中每一行line，處理過的line變成一個list。

### some useful functions

```python
def _gradient(X, Y_label, w, b):
    # This function computes the gradient of cross entropy loss with respect to weight w and bias b.
    y_pred = _f(X, w, b)
    pred_error = Y_label - y_pred
    w_grad = -np.sum(pred_error * X.T, 1)
    b_grad = -np.sum(pred_error)
    return w_grad, b_grad
```

![](https://i.imgur.com/5Caf7Ug.png)

課程中我們找到最佳的w跟b，是找到讓-lnL(w,b)越小越好。要達到目標，就要分別以w跟b對該function微分。最後會得到下面的式子，而b也是同理。

![](https://i.imgur.com/7YL8sOA.png)

### train

這裡的training是用stochastic gradient descent。

## 自我練習

### 流程
* load data
* normalization
* train
    * 參數設定
    * giradient descent
* 計算 loss
* predict
* save file

### 程式(手刻logistic regression版)
```python
import numpy as np
np.random.seed(110)

with open('./data/X_train') as f:
    next(f)
    X_train = np.array([line.strip('\n').split(',')[1:] for line in f], dtype=float)
    
with open('./data/Y_train') as f:
    next(f)
    Y_train = np.array([line.strip('\n').split(',')[1] for line in f], dtype=float)
    
with open('./data/X_test') as f:
    next(f)
    X_test = np.array([line.strip('\n').split(',')[1:] for line in f], dtype=float)

parameter_size = X_train.shape[1]
train_size = X_train.shape[0]
test_size = X_test.shape[0]

# normalize
x_mean = np.mean(X_train, axis=0)
x_std = np.std(X_train, axis=0)

for i in range(train_size):
    X_train[i] = (X_train[i] - x_mean) / (x_std + 1e-8)
for i in range(test_size):
    X_test[i] = (X_test[i] - x_mean) / (x_std + 1e-8)
    
# split into train set & dev set
dev_ratio = 0.3
train_size = int(X_train.shape[0] * (1 - dev_ratio))

X_dev = X_train[train_size:, :]
Y_dev = Y_train[train_size:]
X_train = X_train[:train_size, :]
Y_train = Y_train[:train_size]

print(X_dev.shape)
print(Y_dev.shape)
print(X_train.shape)
print(Y_train.shape)

# training
learning_rate = 0.05
iter_time = 100
step = 1

w = np.zeros((parameter_size, 1))
b = 0

for i in range(iter_time):
    if i%10 == 0:
        print('iter {}'.format(i))
    
    # shuffle
    index = np.arange(train_size)
    np.random.shuffle(index)
    X_train = X_train[index]
    Y_train = Y_train[index]
    
    #sgd
    for j in range(train_size):
        X = X_train[j].reshape(1, -1)
        Y = Y_train[j]
        z = np.dot(X, w) + b
        
        y_pred = 1 / (1 + np.exp(-z))
        pred_error = - (Y - y_pred)
        w_grad = pred_error * X.T
        b_grad = pred_error
        
        w = w - learning_rate * w_grad / np.sqrt(step)
        b = b - learning_rate * b_grad / np.sqrt(step)
        
        step += 1

# calculate train loss
train_loss = 0
for i in range(train_size):
    X = X_train[i]
    Y = Y_train[i]
    z = np.dot(X, w) + b
    y_pred = 1 / (1 + np.exp(-z))
    
    if Y != np.round(y_pred):
        train_loss += 1

print('train loss: {}'.format(train_loss/train_size))

# calculate dev loss
dev_loss = 0
for i in range(X_dev.shape[0]):
    X = X_dev[i]
    Y = Y_dev[i]
    z = np.dot(X, w) + b
    y_pred = 1 / (1 + np.exp(-z))
    
    if Y != np.round(y_pred):
        dev_loss += 1

print('dev loss: {}'.format(dev_loss/X_dev.shape[0]))

ans_y = np.zeros((X_test.shape[0]), dtype=int)
for i in range(X_test.shape[0]):
    X = X_test[i]
    z = np.dot(X, w) + b
    Y_pred = np.round(1 / (1 + np.exp(-z)))
    ans_y[i] = int(Y_pred)

import csv
with open('submit_myself.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    header = ['id', 'label']
    writer.writerow(header)
    
    for i in range(len(ans_y)):
        row = [i, ans_y[i]]
        writer.writerow(row)
```

### 程式(自己用sklearn寫版)
```python
import numpy as np

# read file
with open('./data/X_train') as f:
    content = f.readlines()
    for i,line in enumerate(content):
        if i==0:
            x_train = []
            continue
        line = line.strip('\n').split(',')
        x_train.append(line)
X_train = np.array(x_train, dtype='float')[:, 1:]

with open('./data/Y_train') as f:
    content = f.readlines()
    for i,line in enumerate(content):
        if i==0:
            y_train = []
            continue
        line = line.strip('\n').split(',')
        y_train.append(line)
Y_train = np.array(y_train, dtype='float')[:, 1]

with open('./data/X_test') as f:
    content = f.readlines()
    for i,line in enumerate(content):
        if i==0:
            x_test = []
            continue
        line = line.strip('\n').split(',')
        x_test.append(line)
X_test = np.array(x_test, dtype='float')[:, 1:]

# normalization
x_mean = np.mean(X_train, axis=0)
x_std = np.std(X_train, axis=0)

for i in range(X_train.shape[0]):
    X_train[i] = (X_train[i] - x_mean) / (x_std + 1e-8)
for i in range(X_test.shape[0]):
    X_test[i] = (X_test[i] - x_mean) / (x_std + 1e-8)
    
# split
from sklearn.model_selection import train_test_split
X_train,X_dev,Y_train,Y_dev=train_test_split(X_train, Y_train, test_size=0.25)

# train(LogisticRegression)
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(max_iter=500, random_state=66)
clf.fit(X_train, Y_train)

from sklearn import metrics
print("train accuracy:",metrics.accuracy_score(Y_train, clf.predict(X_train)))
print("dev accuracy:",metrics.accuracy_score(Y_dev, clf.predict(X_dev)))

y_test = clf.predict(X_test).astype(int)
with open('submit_0831_lg.csv', mode='w') as f:
    f.write('id,label\n')
    for i in range(y_test.shape[0]):
        f.write(str(i)+','+str(y_test[i])+'\n')
        
# train(SGDClassifier)
from sklearn.linear_model import SGDClassifier
clf = SGDClassifier(random_state=66, max_iter=500)
clf.fit(X_train, Y_train)
print("train accuracy:",metrics.accuracy_score(Y_train, clf.predict(X_train)))
print("dev accuracy:",metrics.accuracy_score(Y_dev, clf.predict(X_dev)))

with open('submit_0831_sgd.csv', mode='w') as f:
    f.write('id,label\n')
    for i in range(y_test.shape[0]):
        f.write(str(i)+','+str(y_test[i])+'\n')

# train(GaussianMixture)        
from sklearn.mixture import GaussianMixture
clf = GaussianMixture()
clf.fit(X_train)
print("train accuracy:",metrics.accuracy_score(Y_train, clf.predict(X_train)))
print("dev accuracy:",metrics.accuracy_score(Y_dev, clf.predict(X_dev)))

# train(GaussianNB)        
from sklearn.naive_bayes import GaussianNB
clf = GaussianNB()
clf.fit(X_train, Y_train)
print("train accuracy:",metrics.accuracy_score(Y_train, clf.predict(X_train)))
print("dev accuracy:",metrics.accuracy_score(Y_dev, clf.predict(X_dev)))
```

### 總結

課程提供的範例有一點困擾我很久: 在處理多維度的資料時，為了效率與方便，會用內建函式處理。

像是normalize是這樣寫的:

```python
def _normalize(X, train = None, X_mean = None, X_std = None):
    if train:
        X_mean = np.mean(X, axis=0).reshape(1,-1)
        X_std = np.std(X, axis=0).reshape(1,-1)
    X = (X - X_mean) / (X_std + 1e-8)
    return X, X_mean, X_std
```

這樣做是因為np.mean()後得到的numpy array shape 需要調整過，再經由broadcast 把整個X_train直接正規化，寫起來相當直觀易懂，只是真的寫下去卻被背後的邏輯卡很久。

又或者是gradient的部分:

```python
def _gradient(X, Y, w, b):
    y_pred = _f(X, w, b)
    pred_error = Y - y_pred
    
    w_grad = -np.sum(pred_error * X.T, axis = 1)
    b_grad = -np.sum(pred_error)
    return w_grad, b_grad
```

w_grad 跟 b_grad 為什麼這樣寫也不是很好懂，哪怕照著微分後得到的公式去理解也仍然相當頭痛。 像我就相當困惑為何 X 要 transpose? 還有為什麼計算sigmoid時範例用np.matmul，有些人用np.dot? 又或者為何是 np.dot(X, w) ，而不是 np.dot(w, X)呢?

所以我自己嘗試手刻時盡量避免這些雖簡潔卻大有深意的語法，改以寫起來稍嫌複雜，但沒那麼多門道的方法。

normalize的寫法跟作業一一樣。切割訓練集則與範例程式相同，只是沒寫成函式。shuffle的部分也沒寫成函式。

訓練的部分可能沒很符合sgd的精神，等同於把batch_size設為1。

計算loss不是用範例程式裡面的cross entropy loss。

kaggle scores:
* private score: 0.88943
* public score: 0.88914

分數還算不錯，跟用範例程式調參數的結果沒差太多，只是訓練的速度稍慢。


至於從頭到尾自己寫並且用sklearn訓練的版本，基本上就是用自己的方式處理raw data，經過normalization(作業一的寫法)，切割成train set 跟 dev set，然後分別用不同的model訓練，看看結果。

為了不讓每次出來的分數變動太大，在SGDClassifier跟LogisticRegression兩種模型都設定了seed。

最後分數比起上面的不會差很多，但算有略好?只是private score卻比public score高，不確定這樣的結果是否正常。