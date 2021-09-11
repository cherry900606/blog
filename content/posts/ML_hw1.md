---
title: "李宏毅機器學習2020_hw1"
date: 2021-08-08T14:35:19+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - note
---

本篇主要紀錄學習過程中困擾我的部分，以及自己實作的心得。

## 範例code
### Preprocessing
觀察train.csv，取出「數據」的部分，所以前三個column不要。
data中有些為NR，不利於之後處理，通通設值為0。
接著把dataframe轉為numpy array。

```python
data = data.iloc[:, 3:]
data[data == 'NR'] = 0
raw_data = data.to_numpy()
```

### Extract Features (1)
要把資料處理成我們想要的形狀，才能做接下來的訓練。

raw_data的大小是 4320*24。其中4320=12(月)*20(小時)*18(feature)，24為小時。現在我們要將raw_data轉換成dictonary，把同一個月的資料合併，每個index對應到該月資料。
![](https://i.imgur.com/b4oTooq.png)

```python
month_data = {}
for month in range(12):
    sample = np.empty([18, 480]) # 每個月的資料大小
    for day in range(20):
        sample[:, day * 24 : (day + 1) * 24] = raw_data[18 * (20 * month + day) : 18 * (20 * month + day + 1), :]
    month_data[month] = sample
```
### Extract Features (2)
![](https://i.imgur.com/2mS4lbp.png)

接著我們以月為單位，拿到的資料大小是18(feature)*480(=20天 * 24小時)。

為了在現有的資料上生成更多的資料，以每十個小時為單位提取data，每次往右移動一單位。這樣講可能有點抽象，可以想像有一個框框的寬度為10，框住的是新的一筆資料，每次往右滑動一單位。又因為總寬度為480，而框框寬度為10，因此最多能得到471筆資料，因為最後幾筆資料無法框起來。

每次框起來的data有18個row(feature)、10個column(小時)。前九個小時是用來訓練的資料，第十個小時的pm2.5則是答案，分別存在x與y。每個月能生成471筆這樣的data，總共有12個月，因此總共會有417 * 12筆資料，而x的每一筆資料的大小是18 * 9。

注意x的資料是month_data reshape後的，也就是把18個feature的9小時資料變成一個row，類似下圖:
![](https://i.imgur.com/AGu6Yjn.png)
每次拿到新的data就是往下合併，如圖:
![](https://i.imgur.com/7cJuchu.png)

這筆資料該怎麼解讀? 每一個row是各種feature經過9小時的變化數值，同一個column代表相同feature在第幾個小時的數值。
也因此後面normalize的方向是axis=0。

```python
x = np.empty([12 * 471, 18 * 9], dtype = float)
y = np.empty([12 * 471, 1], dtype = float)
for month in range(12):
    for day in range(20):
        for hour in range(24):
            if day == 19 and hour > 14:
                continue
            x[month * 471 + day * 24 + hour, :] = month_data[month][:,day * 24 + hour : day * 24 + hour + 9].reshape(1, -1) #vector dim:18*9 (9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9)
            y[month * 471 + day * 24 + hour, 0] = month_data[month][9, day * 24 + hour + 9] #value
print(x[5651])
print(y.shape)
```
後面範例code的training與testing部分略過。

## 自我練習
以下是我自己寫的code，用sklearn的LinearRegression訓練。

```python
# import libray
%matplotlib inline
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# read file & preprocessing
raw_data = pd.read_csv('train.csv', encoding = 'big5')
raw_data[raw_data == 'NR'] = 0
raw_data = raw_data.iloc[:, 3:]
raw_data = raw_data.to_numpy().astype('float')

# get x_train & y_train
month_data = {}
for month in range(12):
    sample = np.zeros([18, 20*24])
    for day in range(20):
        sample[:, day*24: (day+1)*24] = raw_data[(month*20+day)*18: (month*20+day+1)*18, :]
    month_data[month] = sample
        
y_train = []
for month in range(12):
    for n in range(471):
        sample = month_data[month][:, n:n+9].reshape(1, -1)
        target = month_data[month][9, n+9]
        if n==0 and month == 0:
            x_train = sample
            y_train.append(target)
            continue
        x_train = np.concatenate((x_train, sample), axis = 0)
        y_train.append(target)
 
y_train = np.array(y_train)
y_train = y_train.reshape(-1,1)
x_train = x_train[0:5652] # 發現最後會有一筆重複資料，故刪除

# training
from sklearn.linear_model import LinearRegression
reg = LinearRegression()
reg.fit(x_train, y_train)

# read testing data & preprocessing
test_data = pd.read_csv('test.csv', encoding = 'big5', header=None)
test_data = test_data.iloc[:, 2:]
test_data[test_data == 'NR'] = 0
test_data = test_data.to_numpy().astype(float)

        
for i in range(0, 4320, 18):
        sample = test_data[i:i+18, :].reshape(1, -1)
        if i==0:
            x_test=sample
            continue
        x_test = np.concatenate((x_test, sample), axis=0)
     
# predict
ans_y = reg.predict(x_test)

# write file
import csv
with open('submit_myself.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    header = ['id', 'value']
    writer.writerow(header)
    for i in range(240):
        row = ['id_'+str(i), ans_y[i][0]]
        writer.writerow(row)
```

原本想要偷懶，直接從還沒有合併成一個月一個月的data讀資料，同樣也是一次讀10小時的那種方式。想說不是也能整理成最後的形式嗎，為何要捨近求遠呢?後來才知道主要也是為了增加data數，像是原本的做法只會有3600筆(=15 * 20 * 12)，但範例的作法會有5652筆(=471 * 12)，差了1.5倍，也因此丟到kaggle上分數很差。

private score: 7.51465, public score: 5.46443

後來我又想說: 為何不乾脆把所有資料合併，而不是分成十二個月呢? 重新寫到一半才發現這麼做最後的資料也不過是5760-9筆而已，何況月與月之間的資料不是連續的，因為每個月只有前二十天的數據而已。丟上kaggle後果然分數也比之前差。