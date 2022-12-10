---
title: "論文閱讀 A Hybrid Markov and LSTM Model for Indoor Location Prediction"
date: 2022-01-13T23:00:33+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - 論文
---

## Introduction
隨著終端設備的普及，室內的移動軌跡資料也比以往更加容易取得。這些資料能夠讓店家能針對個別使用者推送個人化訊息，藉由過去的歷史資料來預測使用者接下來的位置，企業將能靈活的提供服務，也能應用在社會安全、智慧運輸、定位服務等場域。

First-order Markov chain(1-MC) 被廣泛運用在時間序列(time-series)資料的預測上，有高度的可詮釋性。可是它假設下一個位置只與當下所在的位置有關，這限制了模型的表現。有學者進一步提出 multi-order Markov chain(k-MC)，它假設下一個位置與前 k 個位置有關，但卻面臨到維度方面的問題(dimension disaster)，因此實際上不太可行。為了解決問題，也有學者用 RNN、LSTM、GRU 等模型嘗試，但雖然比起前者有較高的正確率，卻不容易解釋。

這篇論文的貢獻為：
* 將 multi-order Markov model 轉換為 multiple first-order Markov model 的 multi-step Markov transition probability matrix 解決 dimension disaster。
* 該模型結合 Markov 跟 LSTM 的優勢，能預測序列長度長的資料，也有較高的可解釋性。
* 用實際的室內移動軌跡來訓練 Markov-LSTM model。

## Background
現有的位置預測方法主要可分為兩類：group-based 跟 individual-based model。

**Group-based models** 認為移動的行為會從眾，用其他人的移動軌跡來預測使用者的下一步，找出同的群體間相似的行為。除了能找出群體內使用者在特定情況下的移動模式外，也不需要單一使用者長期的移動紀錄。缺點是模型忽視了相似子群體的存在，只能學個大概。它需要大量的移動資料，而且準確率相對低。

**Individual-based models** 認為每個個體的行為都是獨立的，因此只會用該使用者過去的移動歷史來預測下一個位置。這種方法用來找個體的週期性行為。它的準確率雖然較高，卻需要更多個人資料。

過去的研究多半聚焦於室外的位置預測。

## Preliminaries and Problem Definitions
**Definition 1(Trajectory)**: ![](https://i.imgur.com/iTWuOi4.png)
軌跡包含了了數個有順序的點：
![](https://i.imgur.com/xKGQKun.png)
其中id代表哪位使用者，接下來依序為該點被蒐集的時間、經度、緯度與樓層。

**Definition 2(Slay Point)**:![](https://i.imgur.com/zuBL4ui.png)
表示在某個時間區間內使用者所在的區域，其中分別代表使用者id、平均x、平均y、樓層以及進入與離開該區域的時間。
![](https://i.imgur.com/D4d3cI6.png)

**Definition 3(Location Set)**:![](https://i.imgur.com/DXkvOLB.png)
在這篇論文中代表商店的集合。
![](https://i.imgur.com/RHqYcsg.png)
依序為商店的id、商店的形狀以及樓層。N 是商場中商店的數量。

**Definition 4(Location Sequence)**:![](https://i.imgur.com/Xh1QLof.png)
代表一個使用者所經過的位置與順序。包含一連串在 stay point 的商店。

這篇論文主要就是分析一個使用者的 location sequence，來預測未來的移動位置。假設現在有個使用者 u：
![](https://i.imgur.com/yeMjtwT.png)
式子(1)表示用 location sequence 訓練後的 model 預測，最後得到接下來會拜訪這些商店的機率。
式子(2)表示從機率中選出一個最大的，也就是最有可能拜訪的商店，作為預測下一步位置的結果。

## Methodology
![](https://i.imgur.com/qzFvq0M.png)

方法可以分為四步：location sequence detection, multi-step transition probability matrix definition, adjacent location selection 跟 fusion multiple Markov chains。

首先，由於空間的連續性很難當作 input 給 model，因此要將移動軌跡轉換成跟特定商店有關的 location sequence。再來會定義一種新型的 multi-step Markov transition probability matrix，將 higher-order 的 Markov chain 變成 multiple first-order Markov chain。接著會為每位使用者選出最相鄰的位置。最後會把這些 first-order Markov chain 跟 LSTM model 結合起來，用來預測目標使用者。

### Location Sequence Detection Method
本篇論文用 indoor-STDBSCAN 演算法來找出個體移動軌跡中的 stay point。Indoor-STDBSCAN 演算法將軌跡分為 k 群，也就相當於有 k 個 stay points。
![](https://i.imgur.com/fSPQRLh.png)
其中 sd 是計算空間距離的函式，td 是計算時間距離的函式，等號左側表示集合中的點在時與空都相鄰。

用 indoor-STDBSCAN 演算法得出的 stay point 只有空間上的資訊而沒有語意(semantic)上的意義，因此用 nearest-neighbor search 賦予每個 stay point 意義(semantic)。
![](https://i.imgur.com/WqN3TgM.png)
做法是利用交點法(intersection method)判斷附近商店，以圖為例第二個 stay point 跟商店c的距離為d最靠近，若 d 小於距離門檻(δ)，就讓該 stay point 為使用者拜訪的商店。

### Hybrid Markov-LSTM Model
#### Definition of the Multi-Step Transition Probabilty Matrix
k-MC model 描述一個使用者在不同位置間移動的機率，將 location sequence 中每個位置視為 random variable，用 joint probability 來預測使用者的下一步。
舉例來說，使用者 u 有個長度為 m 的 location sequence![](https://i.imgur.com/mSktgl7.png)，可以用 random variable 的形式表示。![](https://i.imgur.com/gE3e3cU.png)
使用者 u 的 k-order probability transition matrix![](https://i.imgur.com/uSBF8cU.png)，也可以表示為![](https://i.imgur.com/uB25UYb.png)

當 k 增加時，matrix 的增加幅度更加劇烈，讓位置預測不可行。因此，提出 k-step Markov chain。

**Definition 5(1-Step Transition Probability Matrix)**:
1-step transition probability matrix
![](https://i.imgur.com/YX6ztpA.png)
1-order transition probability matrix
![](https://i.imgur.com/KwtdWpD.png)
，兩者都代表使用者 u 從location i 走一步到 location j 的機率。

![](https://i.imgur.com/2j8qraU.png)
分母表示從 location i 走一步到 location j 的距離，分子表示從 location i 走一步到其他 location 的總距離，N 是商場中商店的數量。

**Definition 6(k-Step Transition Probability Matrix)**:
![](https://i.imgur.com/4xWgZvi.png)
![](https://i.imgur.com/SXIZIT8.png)
一個 N * N 的矩陣，表示使用者 u 從 location i 移動 k 步到另一個地點的機率。

![](https://i.imgur.com/yAWMsmt.png)
k-Step Transition Probability Matrix 也可以直接由 1-step transition probability matrix 得來，就是把它乘以 k 次。

透過這種方法，可以讓 k-MC 的機率矩陣大小與 1-MC 相同。
![](https://i.imgur.com/vJr1K1r.png)

#### Selection of the Best Adjacent Locations
Markov-LSTM model 要決定 k 值，也就是要拿前幾個地點來預測下一步。如果 k 值太小，結果就與 1-MC 沒兩樣；如果 k 值太大，模型會變得太複雜跟 overfitting。

透過 cross-validation，找出讓 prediction error 最小的 k。因為每個是使用者都是彼此獨立的個體，所以每個人都 k 值都不同。

#### Fusion of Multiple Markov Models
因為每個人都有自己 k 值，並建立自己的 k first-order Markov model，只是這限制了預測的能力。為了增進模型的預測能力，把 k 個預測的結果跟 LSTM model 結合。

由兩方面來討論模型的優化：

1) 以 Markov model 的角度來看，用多個 1-MC 來取代 k-MC 解決了dimensional disaster。
2) 以 LSTM model 的角度來看，模型不只是從 sequence 找 pattern，而是從 transition probability 的角度來看使用者移動的趨勢。

#### Algorithms and Optimization
使用者的 location sequence 會被分成三部分：historical samples, training samples 跟 test samples。

historical sample 用來建立 k-step transition probobility matrix、training sample 用來定模型的參數，test sample 用來檢測模型預測的表現。

演算法如下：
![](https://i.imgur.com/oZkKRw5.png)

#### Model Complexity Analysis
從空間複雜度來看，k-step transition probability matrix 的大小是 k * N * N，改良後只要 N * N。
從運算的複雜度來看，由演算法的(7)跟(8)可知，算 first-order Markov matrix 比起 k-order 也明顯的複雜度較低。
## Experimental Results and Analysis
### Data Preparation
1) Datasets

實驗的數據主要來自於中國濟南某間商城的 50 位使用者連接 Wi-Fi 定位的資料，定位誤差在三公尺以內。

2) Data Preprocessing

由於行動裝置訊號不穩定，導致數據的異常、錯誤與不合法。

資料集主要有三種類型的 noise：
* 座標異常：超出研究範圍
* 時間異常：從移動軌跡取樣的兩點時間區間為0
* 樓層異常：短時間內在不同樓層間跳動

3) Testbed and test device

### Evaluation Metrics and Comparative Methods
1) Evalutation Metrics

採用 Accuracy, Precision, Recall 跟 F1 作為評估參考。

2) Comparative Methods

MC, HMM, RNN, LSTM,GRU 等模型。

### Variable Estimation
決定模型中的參數數值。

以距離門檻(δ)為例，若該值接近於 0，就只會配對到恰在 stay point 的商店；若該值趨於無限，所有的商店都再範圍內。經分析，分數曲線一開始成長快速，當 δ 大於 6M 後則趨於穩定。在這篇研究中將 δ 設為 4m。

每一位使用者都有自己的 k 值，也就是預測下一步位置所要參考的前幾個位置。由圖示可發現不同使用者最適合的 k 值都不同，也反映出他們是獨立個體。

LSTM 的 hidden states 以 128 為最佳大小。

跟其他模型的表現作比較，發現以 1-MC 的結果最差，與其他模型的分數有明顯斷層，其餘模型的表現接近線性增加，並且以 Markov-LSTM model 表現最好。

## Conclusion and Future Work
室內位置預測的準確在室內定位服務扮演重要角色，尤其是零售業。如果能預測使用者下一個會光顧的商店，同時推送會引起使用者興趣的資訊，不只能提供使用者個人化的購物經驗，更能增加零售業者的獲利。

Markov chain 因為高度的可詮釋性，已經被廣泛運用在位置預測上。然而 k-MC 因為維度的問題不適合 long sequence 的預測。這篇研究提出 Markov-LSTM model 來做室內位置的預測。

首先，定義出 multi-step Markov transition matrix，它將 k-MC 分解為數個 1-MC，解決了 k-MC 的維度問題。接著將數個 1-MC 跟 LSTM 結合來增進預測表現。

實驗使用 45 天來自 50 個使用者的真實室內移動軌跡來驗證該模型的表現。先用 control variable method 得到最適合的參數，再將參數套用到模型上看四種指標：Accuracy, Precision, Recall, F1 的分數。

尚可努力方向：
* 用多元化的資料如 GPS 定位來測試模型
* 與其他預測模型進行廣泛的比較
* 採用更多參數以增進模型在位置預測的表現