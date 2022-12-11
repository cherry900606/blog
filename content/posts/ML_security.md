---
title: "機器學習與資訊安全之應用 期中Project"
date: 2022-12-10T20:28:28+08:00
draft: false
toc: true
images:
tags: 
  - 機器學習
  - 資訊安全
---
## 說明
本篇文章紀錄我在「機器學習與資訊安全之應用」課程的期中作業內容。

期中作業的內容為實作以機器學習中的 few-shot learning 來進行惡意軟體的偵測(malware detection)。須採用 [Dynamic API call sequence visualisation for malware classification](https://ietresearch.onlinelibrary.wiley.com/doi/pdf/10.1049/iet-ifs.2018.5268) 這篇論文特徵提取的方法，並且改以使用 siamese network 來做few-shot learning。

接下來的內容會依序介紹使用的資料集、特徵提取方法、siamese network 的訓練以及結論。

## 資料集介紹
使用的資料集來自實驗室學長姊，是不同 malware family 在執行過程中產生的 system call sequence，這種藉由捕捉惡意程式活動的判別方式屬於動態分析。

單一一筆 sample 的格式如下圖，後續將提取 TIMESTAMP 與 SYSCALL 兩欄的資訊以做分析。
![](https://i.imgur.com/1hZBauo.png)

## 特徵提取
### 論文的方法
論文中的特徵提取方法，是將 API call sequence 依據功能、危險程度分成 16 種類別，並且將惡意程式從開始執行到結束的時段分割為 16 等份，並且依據一個時段內一種類別執行次數給予不同的顏色深淺。藉由將蒐集到的 call sequence 轉換為 image，等同於把惡意程式分類的問題變成圖型的分類，其想法是同一種 family 的惡意程式會為了規避被偵測而略做修改，然而整體行為模式還是相似的，因此可想而知不同的 malware family 所形成的圖像會不太一樣。

![](https://i.imgur.com/KrY1Y7T.png)

### 我的方法
在課程作業的實作中，我使用的是 system call 而非 API call，加上論文作者也沒給不同 API call 分類底下各自有哪些類型，因此需要自行替 system call 做分類。

SYSCALL 指的是 system call，屬於底層作業系統中所提供的介面。相對的，API (Application Program Interface) 是為了方便使用者而開法的介面，使用者只須要知道要呼叫哪些 API 以進行開發，無須了解該 API 會呼叫哪些 system call。

我的方法是在搜尋各種 system call 的含意時產生的靈感。以 execve 這個 system call 為例，[linux man-pages](https://man7.org/linux/man-pages/man2/execve.2.html) 中的 SYNOPSIS 小節提到需要引入 header 檔案 unistd.h。因此嘗試以是否引入同樣的 header 作為分類依據，並選出 16 種類別，其中難以分類的放入第 16 種類別 other。

論文中的 image 維度 16 * 16，代表分成 16 種類別，以及將程式的運行時間切成 16 等分。在此也是選擇同樣的 image 大小，上色的方式也比照論文提供的 hex 對照表。

接著對各種 malware family 作圖，觀察圖像是否有規律存在。經觀察發現，每種 malware family 都有屬於自己的特殊的 pattern，分別如下：
| Malware family | Image                                |
| -------------- | ------------------------------------ |
| Mirai          | ![](https://i.imgur.com/FpyPVW7.png) |
| Bashlite       | ![](https://i.imgur.com/8yOSrhU.png)|
| Tsunami        | ![](https://i.imgur.com/V0kHtwS.png)|
| Android        | ![](https://i.imgur.com/PcSQcAh.png)|
| Dofloo         | ![](https://i.imgur.com/3XfCQV9.png)|
| Unknown        | ![](https://i.imgur.com/sPkbAOA.png)|
| Hajime         | ![](https://i.imgur.com/dJtgPjZ.png) |
| Xorddos          | ![](https://i.imgur.com/dTdqXNg.png) |

然而，並不是同個 malware family 底下的圖片都遵循這樣的 pattern，背後的原因仍有待了解。推測是與每個程式是否有完整執行到有關，有些檔案因為惡意程式沒有完整的執行，因此 system call 數量不足，以至於呈現出來的圖片會有偏差。

最後選擇以其中的三個 family：Mirai、Android 與 Unknown 來做訓練，各取 10 張圖片，一半作為訓練集，另一半則作為測試集，屬於 3 way 5 shot。經測試訓練集的正確率為 0.71，測試集的正確率為 0.66。

## 訓練 siamese network

Siamese network，又稱為孿生網路，是一種特殊的神經網路架構。它是由兩個相同結構、共享權重的神經網路所構成，用以比較兩個輸入間的相似程度。

在 GitHub 上的[一個專案](https://github.com/aspamers/siamese)提供 Siamese network 的 API，可以很容易的設定模型架構以及呼叫該模型，因此在這份作業我們以其為基礎加上部分修改，並套用在作業的資料集上。
模型架構示意圖：
![](https://i.imgur.com/hdYTEpx.png)

其中，base model 與 head model 的細節如下：

![](https://i.imgur.com/A0ny7Jk.png)

## 總結
1. 嘗試以 system call sequence，而不是 API call sequence 來轉換成 image 以及後續的影像辨識
2. 轉換成 image 時需要把 system call 做分類，我們以 system call 所引入的 header 作為分類的依據



