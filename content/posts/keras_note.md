---
title: "Keras 筆記"
date: 2021-09-20T22:34:47+08:00
draft: false
toc: true
images:
tags: 
  - 深度學習
---

## 常用import
```python
from keras.models import Sequential
from keras.layers.core import Dense, Dropout, Activation
from keras.layers import Conv2D, MaxPooling2D, Flatten, Conv1D, MaxPooling1D
from keras.utils import np_utils
from tensorflow.keras.optimizers import SGD, Adam
```

## define model

```python
model = Sequential()

model.add(Dense(input_dim=28 * 28, units=689, activation='relu'))
model.add(Dropout(0.7))
model.add(Dense(units=689, activation='relu'))
model.add(Dropout(0.7))
model.add(Dense(units=10, activation='softmax'))
```
input layer 需要設定input_dim或者input_shape，依據data本身的shape設定，之後的layer則不用。units表示每層的神經元數目。
最後一層的units需與預測種類的數目一致，以MNIST為例，因為有0~9共10個數字，因此要設為10。

## goodness of function

```python
model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
```
## 訓練

```python
model.fit(X_train, Y_train, batch_size=100, epochs=20)
```

## testing & predict

```python
result_train = model.evaluate(X_train, Y_train)
print('Train Loss:', result_train[0])
print('Train Acc:', result_train[1])
```

## 其他

以MNIST為例，其label是以十進位表示，必須轉換表達方式。
```python
Y_train = np_utils.to_categorical(Y_train, 10)
```

使用conv(卷積層)時，可能會出現input_shape方面的錯誤。以Conv2D為例，要求的input shape為3維(寬:高:層數)，此時reshape成相應格式即可。
以MNIST為例，用mnist.load_data()讀取資料後:
```python
X_train = X_train.reshape(X_train.shape[0], 28, 28, 1)
X_test = X_test.reshape(X_test.shape[0], 28, 28, 1)
```
或者使用Conv1D，我對syscall得到的X_train與X_test進行處理:
```python
X_train = X_train.reshape(X_train.shape[0], 28, 28, 1)
X_test = X_test.reshape(X_test.shape[0], 28, 28, 1)
```
總之，就是依需求調整shape。

最後放上syscall的keras+CNN的部分code，方便以後參考:
```python
model = Sequential()

model.add(Conv1D(25, kernel_size=3, activation='relu', input_shape=(173, 1)))
model.add(MaxPooling1D(pool_size=2))
model.add(Conv1D(25, kernel_size=3, activation='relu'))
model.add(MaxPooling1D(pool_size=2))
model.add(Conv1D(50, kernel_size=3, activation='relu'))
model.add(MaxPooling1D(pool_size=2))


model.add(Flatten())

model.add(Dense(units=250, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(units=100, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(units=100, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(units=8, activation='softmax'))
```