---
title: "後門攻擊"
date: 2022-02-14T13:57:49+08:00
draft: false
toc: true
images:
tags: 
  - 後門攻擊
---

## 簡介
深度學習會從訓練資料中提取特徵，因此若攻擊者想讓深度學習模型會在特定情況下誤判，可以在部分的訓練資料中加入觸發器(trigger)，並將標籤設為目標類別。如以一來，當訓練好的模型遇到含有觸發器的輸入時，就會被導向攻擊者期望的目標類別，而非原本的預測結果。除此之外，如果該模型遇到正常輸入，其表現也應與未被攻擊的模型沒差不多。

## 方法
在一部份資料加上觸發器，同時更動標籤類別。

```python
# 在 trainset 前 1000 筆資料植入trigger，並導向 label 4
for i in range(1000):
  cv2.rectangle(trainset.data[i].numpy(),(0,0),(2,2),(255,255,255),-1)
  #cv2_imshow(trainset.data[i].numpy())
trainset.targets[:1000] = 4
```
```python
# 查看圖片
plt.imshow(trainset.data[0], cmap='gray')
plt.title('%i' % trainset.targets[1])
plt.show()
```
![](https://i.imgur.com/oR12L4D.png)

可以看到圖片的左上角有白色方形，就是後來加入的觸發器。

接著再丟進模型訓練即可。

比較完整的 code 如下：
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import transforms, datasets

trainset = datasets.MNIST('', train=True, download=True, 
                       transform=transforms.Compose([
                                transforms.ToTensor()
                            ]))
testset = datasets.MNIST('', train=False, download=True, 
                       transform=transforms.Compose([
                                transforms.ToTensor()
                            ]))

trainloader  = torch.utils.data.DataLoader(trainset, batch_size=100, shuffle=True, pin_memory=True)
testloader  = torch.utils.data.DataLoader(testset, batch_size=100, shuffle=False, pin_memory=True)

class Net(nn.Module):
    def __init__(self):
        super().__init__()

        self.conv = nn.Sequential(
            nn.Conv2d(1, 64, 5),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(64, 32, 3),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
        )
        self.fc = nn.Sequential(
            nn.Linear(32 * 5 * 5, 128),
            nn.ReLU(),
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Linear(64, 64),
            nn.ReLU(),
            nn.Linear(64, 64),
        )
        
    def forward(self, x):
        x = self.conv(x)
        x = x.view(-1, 32 * 5 * 5)
        x = self.fc(x)
  
        return F.log_softmax(x, dim=1)
      
net = Net() # inital network 

optimizer = optim.Adam(net.parameters(), lr=0.001)  # create a Adam optimizer

def train(dataloader, model, optimizer):
  model.train()
  running_loss = 0.0
  for i, data in enumerate(dataloader):
    X, y = data

    # training process
    optimizer.zero_grad()    # clear the gradient calculated previously
    predicted = model(X)  # put the mini-batch training data to Nerual Network, and get the predicted labels
    loss = F.nll_loss(predicted, y)  # compare the predicted labels with ground-truth labels
    loss.backward()      # compute the gradient
    optimizer.step()     # optimize the network
    running_loss += loss.item()
    if i % 100 == 99:    # print every 1000 mini-batches

      print('[%d, %5d] loss: %.3f ,acc: %.2f' %
            (epoch + 1, i + 1, running_loss / 100,(torch.argmax(predicted, 1)==y).sum().item()/100))
      running_loss = 0.0

def test(dataloader, model):
  model.eval()
  correct = 0
  total = 0
  with torch.no_grad():
      for data in dataloader:
          X, y = data
          output = model(X)
          correct += (torch.argmax(output, dim=1) == y).sum().item()
          total += y.size(0)

  print(f'testing data Accuracy: {correct}/{total} = {round(correct/total, 3)}')
            
import cv2
import numpy as np
from google.colab.patches import cv2_imshow
import matplotlib.pyplot as plt

# 在 trainset 前 1000 筆資料植入trigger，並導向 label 4
for i in range(1000):
  cv2.rectangle(trainset.data[i].numpy(),(0,0),(2,2),(255,255,255),-1)
  #cv2_imshow(trainset.data[i].numpy())
trainset.targets[:1000] = 4

# train 5 iter
for epoch in range(5):
  print('epoch:',epoch)
  train(trainloader, net, optimizer)
    
# testloader 還沒加 trigger 時的正確率
test(trainloader, net)
test(testloader, net)

# 用來比較 testset 前幾筆有加入trigger 的預測結果
def t(dataloader, model):
  model.eval()
  correct = 0
  total = 0
  with torch.no_grad():
      for i, data in enumerate(dataloader):
          X, y = data
          output = model(X)
          print(y)
          print(torch.argmax(output, dim=1))
          # correct += (torch.argmax(output, dim=1) == y).sum().item()
          # total += y.size(0)

          break

  #print(f'testing data Accuracy: {correct}/{total} = {round(correct/total, 3)}')

# 在 testset 前 10 筆資料植入trigger
for i in range(10):
  cv2.rectangle(testset.data[i].numpy(),(0,0),(2,2),(255,255,255),-1)
  cv2_imshow(testset.data[i].numpy())
    
t(testloader, net)
```

