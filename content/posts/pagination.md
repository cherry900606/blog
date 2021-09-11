---
title: "在hugo hermit主題增加分頁功能"
date: 2021-09-11T11:53:57+08:00
draft: false
toc: false
images:
tags: 
  - blog
---

根據[這篇文章](https://fffou.com/post/2020-05-14/)的說明，可以在hugo的hermit主題自行增加分頁功能，不然hermit本身不會換頁。

我在「在對應的CSS樣式文件中添加以下」的步驟遇上困難，不知道具體該怎麼做。

方法就是在/layouts/partials中直接新增一個pagination.css的檔案，把那段code貼上去就好了。

不過似乎還是有錯誤，導致理想中的效果無法呈現，這部分待以後研究。