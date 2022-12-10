---
title: "不定長度數字序列的分類問題"
date: 2021-12-09T22:11:58+08:00
draft: false
toc: false
images:
tags: 
  - untagged
---

## 題目敘述

從 binary malware file 轉成 number sequence，長度不一。最長的數字序列長度為14231552、最短為32。訓練資料共有3604筆，label數為28種 malware。

## 前情提要
由於 sequence 的長度變動相當大，除了長度為50176的佔了大多數，其餘分布相當零散。如果全部 padding 到最大長度，在 colab 跑不起來；如果截長補短到325*325，用 CNN 訓練，正確率約在9%左右。

此篇嘗試另一種方法──以 markcov chain 這組關鍵字出發。

Markcov chain 是一種模型，描繪狀態與狀態間的轉變關係。也就是說，它只看「當下的狀態」與「新的input」來決定下一步是哪個狀態。這樣的方式運用在此例時，恰好能幫助擺脫 sequence 長度的限制。

LSTM 則是在此概念上更進一步，能夠記憶近期的資訊，並且遺忘部分資訊。

## 方法嘗試

此節整理網路上的資料。

[LSTM - Sequences with different num of time steps](https://github.com/keras-team/keras/issues/85)

1. padding zeros 到同樣長度
2. batch_size=1，也就是一次只餵一筆資料進model
```python
model.add(LSTM(512, return_sequences=True, input_shape=(None, len(chars))))
```
這裡的input_shape=(timestep, feature dim)。因為長度不固定，timestep=None；feature dim則是feature的數量。
3. bucketing and padding。讓每一個bucket都是長度相近的seq，最後同個bucket的長度補到一樣。用 train_on_batch。
