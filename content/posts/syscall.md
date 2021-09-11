---
title: "syscall"
date: 2021-09-05T14:25:03+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - note
---

## 目標

閱讀README.txt，裡面提到用label.txt內有列出的檔案即可。

label.txt的內容分為兩部分: 檔案名稱與對應的malware類型。

資料夾strace19000內則有相當多的檔案，不過其中只有檔名出現在label.txt的檔案才會被用到。每個檔案內容由不同的system call組成，檔案長度不一。

本次練習的目標是根據有被label的檔案的system call，預測對應的malware類型。

## 想法

建立一個二維陣列的表格，紀錄每一個檔案中不同的system call出現的次數。根據這個表格訓練模型。

因此，我們要:

1. 找出所有出現過的system call種類
2. 統計每個檔案出現過哪些system call，以及出現的次數

## 程式

### 讀檔與前處理
```python
import pandas as pd
import numpy as np

# read file
with open('./syscall/label.txt', mode='r') as file:
    data = np.array([line.strip('\n').split(' ') for line in file], dtype=str)
    label_file = data[:, 0]
    target = data[:, 1]
```
先把label.txt通通讀進來，用空白做切割。之後再分別放進label_file與target。

```python
# explore data
unique, counts = np.unique(target, return_counts=True)
dict(zip(unique, counts))
```
看看target總共有哪些種類，以及每個種類的個數。

```python
# convert label into file name
labeled_file_name = np.array(['syscall_'+line+'-straceLog.tx' for line in label_file], 
                             dtype=str)
```
因為strace19000中的檔名跟label.txt裡面的不太一樣，為了方便先轉成對應格式。

```python
syscall_set = set()
data_size = len(labeled_file_name) #2000
for i in range(len(labeled_file_name)):
    path = './syscall/strace19000/'+labeled_file_name[i]
    with open(path) as file:
        syscall = np.array([line.strip('\n') for line in file])
        syscall_set.update(set(syscall))
        if i%100 == 0:
            print('{}/{}'.format(i, len(labeled_file_name)))
        
        #先讀前1000筆看看?
        if i==data_size:
            break
```
這裡是把strace19000中有被label的檔案讀一遍，目的是統計所有出現過的system call種類，把它們以set記錄起來。
因為一次讀完所有被label的檔案會有點久，設data_size可以先用一部份資料訓練看看。

```python
syscall_list = list(syscall_set)
table = np.zeros((data_size, len(syscall_list)))

for i in range(len(labeled_file_name)):
    path = './syscall/strace19000/'+labeled_file_name[i]
    with open(path) as file:
        # 先讀前1000筆看看?
        if i==data_size:
            break
        
        syscall = np.array([line.strip('\n') for line in file])
        unique, counts = np.unique(syscall, return_counts=True)
        sys_dict = dict(zip(unique, counts))
        
        # fill table
        for j in range(len(syscall_list)):
            if syscall_list[j] in sys_dict:
                table[i][j] = sys_dict[syscall_list[j]]

        if i%100 == 0:
            print('{}/{}'.format(i, len(labeled_file_name)))
```
有了所有的system call種類後，就可以來統計數字了。
table就是那張表格，row數代表有多少個labeled file，column代表有多少種system call。

每讀一個檔案，就把所有的syscall轉成dictionary，裡面有類型跟出現次數。接著拿syscall_list遍歷，如果該種類在這個file出現過，那就更新table數字，沒有的話就不用管了，因為一開始宣告的就是zero numpy array。

### 訓練與測試

接下來要做的，就是切割成訓練集跟測試集！

為了讓參數相同的情況下結果一致，所以設定radom_state。

```python
# split training set and testing set
from sklearn.model_selection import train_test_split
X_train, X_test, Y_train, Y_test = train_test_split(table, target[:data_size], 
                                                    test_size=0.25, random_state=66)
```

分別用logistic regression, SVM, random forest訓練。

### 一般
```python
# training and testing
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(random_state=66)
clf.fit(X_train, Y_train)
from sklearn.metrics import accuracy_score
print('---logistic regression---')
print('train acc:', accuracy_score(Y_train, clf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, clf.predict(X_test)))

from sklearn.svm import SVC
svm = SVC(random_state=66)
svm.fit(X_train, Y_train)
print('---SVM---')
print('train acc:', accuracy_score(Y_train, svm.predict(X_train)))
print('test acc:', accuracy_score(Y_test, svm.predict(X_test)))

from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(random_state=66)
rf.fit(X_train, Y_train)
print('---random forest---')
print('train acc:', accuracy_score(Y_train, rf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, rf.predict(X_test)))
```
分數如下:
```
---logistic regression---
train acc: 0.8646019278828989
test acc: 0.8704149933065596
---SVM---
train acc: 0.8961977865048197
test acc: 0.8966532797858099
---random forest---
train acc: 0.9991967154587648
test acc: 0.9775100401606426
```

發現random forest的分數明顯比其他兩者好上太多。但如果對資料再做些處理呢?

### feature selection

有些system call在所有file出現過的次數很少，如果把它們移除，或許能提升分數。

```python
# find the features that ouccrence times are less than 100 in all label file
print(table.sum(axis=0)[table.sum(axis=0)<10])
print(np.where(table.sum(axis=0)<10))
drop_feature = np.where(table.sum(axis=0)<10)
new_table = np.delete(table, drop_feature, axis=1)
```
把次數小於10次的那幾個column移除。

```python
X_train, X_test, Y_train, Y_test = train_test_split(new_table, target[:data_size], 
                                                    test_size=0.25, random_state=66)

from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(random_state=66)
clf.fit(X_train, Y_train)
from sklearn.metrics import accuracy_score
print('---logistic regression---')
print('train acc:', accuracy_score(Y_train, clf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, clf.predict(X_test)))

from sklearn.svm import SVC
svm = SVC(random_state=66)
svm.fit(X_train, Y_train)
print('---SVM---')
print('train acc:', accuracy_score(Y_train, svm.predict(X_train)))
print('test acc:', accuracy_score(Y_test, svm.predict(X_test)))

from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(random_state=66)
rf.fit(X_train, Y_train)
print('---random forest---')
print('train acc:', accuracy_score(Y_train, rf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, rf.predict(X_test)))
```
分數如下:
```
---logistic regression---
train acc: 0.8643341663691538
test acc: 0.8698795180722891
---SVM---
train acc: 0.8961977865048197
test acc: 0.8966532797858099
---random forest---
train acc: 0.9991967154587648
test acc: 0.9772423025435074
```
結果......好像沒什麼幫助。

### normalization

使用z-score normalization。

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(table)
new_table = scaler.transform(table)

X_train, X_test, Y_train, Y_test = train_test_split(new_table, target[:data_size], 
                                                    test_size=0.25, random_state=66)

from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(random_state=66)
clf.fit(X_train, Y_train)
from sklearn.metrics import accuracy_score
print('---logistic regression---')
print('train acc:', accuracy_score(Y_train, clf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, clf.predict(X_test)))

from sklearn.svm import SVC
svm = SVC(random_state=66)
svm.fit(X_train, Y_train)
print('---SVM---')
print('train acc:', accuracy_score(Y_train, svm.predict(X_train)))
print('test acc:', accuracy_score(Y_test, svm.predict(X_test)))

from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(random_state=66)
rf.fit(X_train, Y_train)
print('---random forest---')
print('train acc:', accuracy_score(Y_train, rf.predict(X_train)))
print('test acc:', accuracy_score(Y_test, rf.predict(X_test)))
```
分數如下:
```
---logistic regression---
train acc: 0.958853980721171
test acc: 0.9544846050870147
---SVM---
train acc: 0.9378793288111389
test acc: 0.9349397590361446
---random forest---
train acc: 0.9991967154587648
test acc: 0.9775100401606426
```
雖然random forest的結果沒變，但是前兩者的分數卻大大提升了。