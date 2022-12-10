---
title: "Word2vec 筆記"
date: 2021-10-02T17:42:15+08:00
draft: true
toc: true
images:
tags: 
  - note
---

## One-hot encoding

假設一個字串 s = "I love reading"，經過one-hot encoding，可能得到dict為:{"I":1, "love":2, "reading":3}

字串的每個字會被轉換成向量:

I: [1,0,0]

reading: [0,0,1]

但有以下缺點:
* 矩陣稀疏
* 過高維度
* 無法保留語意

## Embedding

one-hot encoding會得到向量，只是向量的維度大太。Embedding同樣是將字詞映射到向量空間表示，但維度沒那麼大。也因此可以透過轉換得到的向量，來分析字詞間的關係。

## 程式

### Tokenizer
```python
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences

docs = ["The cat sat on the mat.", "I love green eggs and ham."]

tokenizer = Tokenizer(num_words=8)
tokenizer.fit_on_texts(s)
word_index = tokenizer.word_index

sequences = tokenizer.texts_to_sequences(s)
data = pad_sequences(sequences, maxlen=8)
```
Tokenizer會透過fit_on_texts將輸入的字串編成dict，其中參數num_words表示只包含最常出現的特定種字詞。

word_index可以看到dict。

把字串經由dict變成以數字形式呈現，使用text_to_sequences。

補齊至特定長度，使用pad_sequences。