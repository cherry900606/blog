---
title: "在hugo hermit主題加入留言功能"
date: 2021-08-11T18:02:29+08:00
draft: false
toc: true
images:
tags: 
  - note
  - blog
---

## Disqus
在[hermit](https://github.com/Track3/hermit)主題加入留言功能非常簡單。

首先，照著[這篇](https://sunnyday0932.github.io/2020/hugo%E5%8A%A0%E5%85%A5%E7%95%99%E8%A8%80%E8%A7%80%E7%9C%8B%E4%BA%BA%E6%95%B8/)到第七步後，由於hermit主題已經有寫好相關功能，push上去後應該就成功了，只是無法在本地端看到結果。

目前遇到的問題是部落格名稱無法正常顯示，如下圖。"Cody Crnkovich's Blog"的部分應該要與我在disqus填寫的website name一致才對。
![](https://i.imgur.com/g3M2KY7.png)

## Utterance

雖然無法解決上述問題，不過在尋找解答的過程中發現了utterance。

照著[這篇文章](https://www.jkg.tw/p3350/)設定，把產生的程式碼貼到 ./partials/comments.html。貼上之把原本的內容先清掉。

如果要刪除留言就到repo，把issue delete掉就沒了！