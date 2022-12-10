---
title: "Adversarial attack(對抗式攻擊)"
date: 2022-06-05T18:32:56+08:00
draft: false
toc: true
images:
tags: 
  - note
---

對抗式攻擊(adversarial attack)是透過在input加上細微的擾動，讓模型做出錯誤的判斷，被攻擊的模型卻可能對錯誤的預測有高度信心。

可以用黑箱攻擊(Black-box attacks)或白箱攻擊。前者對於模型僅有有限的了解，像是知道訓練的步驟或模型架構，但也可能沒有任何關於模型的資訊；後者是對目標模型有全面性的認識，知道模型的參數數值、架構等等。

對抗式攻擊的目標是找到微小的擾動值，當該原本的sample加上擾動值後，會讓classifier預測出錯誤的label。

除了電腦視覺(computer version)領域外，malware detector也有遭受對抗式攻擊的可能。不過相較於圖像的input是連續的數值，malware 的 input會是離散的。此外，視覺上的相似性會被運作行為相似性取代。

對抗式攻擊也能應用在現實生活中，並且許多文獻已證實不同的深度神經網路都會被影響。由一個模型得到的對抗例(adversarial example)，拿到不同的神經網路也有效。而這個領域還有待開發。