---
title: "Pytorch MNIST 筆記"
date: 2021-08-23T14:56:12+08:00
draft: false
toc: true
images:
tags: 
  - pytorch
  - 機器學習
  - note
---

## 流程

1. load library
2. read data and pre-processing(set parameters, create dataloader)
3. define network structure(set model, set loss function, set optimizer)
4. training
    * gradient 歸零
    * predict
    * 計算loss
    * backward
    * 更新step
5. testing

## 程式
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import datasets, transforms
import torch.utils.data as data
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np

# setting & preprocceing data
BATCH_SIZE = 64
EPOCHS = 1

transform = transforms.Compose([
    transforms.ToTensor(),
])

train_dataset = datasets.MNIST(root='.', train=True, transform=transform, download=True)
test_dataset = datasets.MNIST(root='.', train=False, transform=transform, download=True)

train_loader = data.DataLoader(train_dataset, batch_size=BATCH_SIZE, shuffle=True)
test_loader = data.DataLoader(test_dataset, batch_size=BATCH_SIZE, shuffle=False)

# define network structure
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.cnn = nn.Sequential(
            nn.Conv2d(1, 32, 3),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2, 2, 0)
        )
        self.fc = nn.Sequential(
            nn.Linear(32*13*13, 1024),
            nn.ReLU(),
            nn.Linear(1024, 256),
            nn.ReLU(),
            nn.Linear(256, 10)
        )
    
    def forward(self, x):
        x = self.cnn(x)
        x = torch.flatten(x, 1)
        x = self. fc(x)
        output = F.log_softmax(x, dim=1)
        return output
    
model = Net()
loss = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters())

# train
train_loss = []
for epoch in range(EPOCHS):
    model.train()
    for i, (data, target) in enumerate(train_loader):
        optimizer.zero_grad()
        train_pred = model(data)
        batch_loss = loss(train_pred, target)
        batch_loss.backward()
        optimizer.step()
                   
        if i%10 == 0:
            print("loss: {:.6f} [{}/{}]".format(batch_loss.item(), 
                                            (i+1) * len(data),
                                            len(train_loader.dataset)))
            train_loss.append(batch_loss.item())

    
# test
test_loss = 0
test_acc = 0
model.eval()
with torch.no_grad():
    for data, target in test_loader:
        test_pred = model(data)
        test_loss += loss(test_pred, target).item()
        
        pred = test_pred.argmax(dim=1, keepdim=True)
        test_acc += (pred == target.view_as(pred)).sum().item()
        
    test_loss /= len(test_loader.dataset)

    # Log testing info
    print('\nTest set: Average loss: {:.4f}, Accuracy: {}/{} ({:.0f}%)\n'.format(
        test_loss, test_acc, len(test_loader.dataset),
        100. * test_acc / len(test_loader.dataset)))
```

## 學習資源
[PyTorch官方Tutorials Quickstart](https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html)

[Day 9 / PyTorch 簡介 / PyTorch 入門（二） —— MNIST 手寫數字辨識](https://ithelp.ithome.com.tw/articles/10243145)

