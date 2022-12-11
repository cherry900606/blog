---
title: "機器學習與資訊安全之應用 期末Project"
date: 2022-12-10T22:40:57+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - 資訊安全
---
## 說明
本篇文章紀錄我在「機器學習與資訊安全之應用」課程的期末作業內容。

期末作業的要求是實作在 malware detection 上的對抗式攻擊(adversarial attack)，並且應用在 siamese network 上。我們選擇實作 [Evading API Call Sequence Based
Malware Classifiers](https://link.springer.com/content/pdf/10.1007/978-3-030-41579-2_2.pdf) 中的 Feature Level Evasion 方法。

## 攻擊手法
### 對抗式攻擊
對抗式攻擊，是藉由在輸入中加入雜訊以愚弄 AI 模型並且產生非預期的預測結果。

例如在經典的 MNIST 手寫數字資料集中，讓 AI 模型遇到輸入的左上角是白點的圖型，即判斷為是數字4，我曾在[這篇文章](https://cherry900606.github.io/posts/backdoor/)實作過；又或者是在 EMBER 這個蒐集惡意程式各種特徵並向量化的資料集，藉由找出 benign sample 中較有影響力的 trigger 並嵌入 malware sample 中，即可讓訓練好的模型做出誤判，我亦曾在[這篇文章](https://cherry900606.github.io/posts/implement_backdoor_by_shap/)實作過。更生活化的例子，是在路標上貼上特定符號，讓自駕車做出錯誤的判斷，這樣的攻擊若成功，造成的後果十分嚴重。
### 論文的方法
![](https://i.imgur.com/RhUe9gP.png)

論文提出的是一種 iterative 的方法，針對使用 API call sequence 來偵測惡意程式的 AI 模型。它屬於黑箱攻擊，也就是不需要知道模型本身的細節就能進行攻擊。

在這個方法中，每次都會將現階段的 API call sequence 丟進事先訓練好模型，若被判斷為 malware 就會繼續修改。每次的修改會加入一個 API call，並且是找出某隨機位置之下，要搭配哪個 API call 所形成的 2-gram，能夠符合「出現在 benign sample 的比例 - 出現在 malware sample 的比例」最大的那組解。

### 我的方法
我仿照論文的寫法，提取前 300 個 API call sequence 來做訓練，並取 50 個 malware sample 嘗試生成對抗式樣本。

由於手上的資料集中，benign sample 的 system call 數量都遠小於 300，決定取兩個 malware family，一個當 malware 一個當 benign，看能不能騙過 detector。

現階段的實驗中，我將 iterative 的次數限制在最多 50 次沒成功就會停止以節省時間，最後有 45 個 sample 順利跑完，其中有 23 個 sample 被辨認為 benign，成功率算是很高。

## 應用在 siamese network
待實作。