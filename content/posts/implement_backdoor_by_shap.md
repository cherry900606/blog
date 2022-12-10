---
title: "實作後門攻擊"
date: 2022-06-07T12:13:23+08:00
draft: false
toc: true
images:
tags: 
  - untagged
---

## 簡介
這次我用在論文閱讀中學到的方法練習實作後門攻擊。使用的是部份的 EMBER 資料集，包含 9999 筆資料，其中前 5000 筆的 label 是 0 (benign),其餘是 1 (malware)。

我先將讀入的資料做簡單的預處理(shuffle + 正規化)，接著切割成 train, test set 後餵給 model，再用得到的 model 跟 shap 套件得到 shap 值。

經過 feature selection 跟 value selection 後會得到 trigger。因為是 clean label attack，把該 trigger 藏進部分的 benign sample 來訓練模型，最後得到的就是 backdoored model。

## feature selection
n 用來指定 trigger size 多大。
選擇 feature 的依據是找出 feature 中 shap 的絕對值總和最大的 n 個，代表影響力越大。

```python
def LargeAbsSHAP(s, feats, n, label=0):
    s = np.abs(shap_values[label]).sum(0)
    larget_shap_index = s.argsort()[-n:]
    print(larget_shap_index)
    print(s[larget_shap_index])
    return larget_shap_index
selected_feature = LargeAbsSHAP(shap_values, np.arange(X_train.shape[1]), 16)
```

## value selection
我實作了兩個論文中提到的選擇 vaule 的方式，分別為 CountSHAP 跟 MinPopulation。

CountSHAP 是找出每個 feature 底下，依據所有 value 數值的出現次數，以及對應到 shap 值得總和，找出最小的 n 個。
MinPopulation 是直接找出出現次數最少的 n 個。
```python
import collections
def CountSHAP(s, x, f, label=0):
    result = []
    for _f in f:
        mini = 100000 # 根據公式算出來的值
        min_arg = -1 # 公式算出來最小值的那個 feature value 是啥
        c = collections.Counter(np.array(x)[:,_f]).most_common() # label 0, all sample & _f feature
        for _c in c:
            now = (1/_c[1]) + s[0][[idx for idx, x in enumerate(x[:1000,_f]) if x == _c[0]],_f].sum()
            if now < mini:
                mini = now
                min_arg = _c[0]
        result.append((_f, min_arg))
        #print(mini, min_arg)
    print(result)
    return result

def MinPopulation(s, f, label=0):
    result = []
    for _f in f:
        c = collections.Counter(np.array(s)[0][:,_f]).most_common()[-1][0]
        result.append((_f, c))
    print(result)
    return result
selected_value = CountSHAP(shap_values_dc, X_train, selected_feature)
```

## 結果
我用了幾種模型來檢驗結果，並用三種指標 Acc(F, X), Acc(Fb, X) 跟 Acc(Fb, Xb) 作為標準。

其中 F 代表正常的 model， Fb 代表有後門的 model； X 代表正常的 malware sample，Xb 代表全部加上 trigger 的 malware sample。

Acc(F, X) 看出正常情況下，模型對 malware sample 的檢測率，Acc(Fb, X) 看有後門的模型對正常 malware sample 的正確率有沒有降，Acc(Fb, Xb) 看有多少比例的 malware sample 在加上 trigger 後成功偽裝成 benign sample。 

(trigger size=16, poison rate=1%)

```
* LGBMClassifier
Acc(F, X): 0.982
Acc(Fb, X): 0.981
Acc(Fb, Xb): 0.0

* XGBClassifier
Acc(F, X): 0.981
Acc(Fb, X): 0.983
Acc(Fb, Xb): 0.081
0.983/0.081 = 12.3倍

* RandomForestClassifier
Acc(F, X): 0.949
Acc(Fb, X): 0.947
Acc(Fb, Xb): 0.943

* DecisionTreeClassifier
Acc(F, X): 0.968
Acc(Fb, X): 0.967
Acc(Fb, Xb): 0.549
0.967/0.549 = 1.8倍

* LogisticRegression
Acc(F, X): 0.943
Acc(Fb, X): 0.944
Acc(Fb, Xb): 0.291
0.944/0.291 = 3.2倍
```