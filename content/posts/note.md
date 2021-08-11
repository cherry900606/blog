---
title: "Hugo note"
date: 2021-08-05T12:17:49+08:00
draft: false
toc: true
images:
tags: 
  - note
---

## 架站教學
[Hugo靜態網站快速入門](https://aishuafei.com/hugo-getting-started/)

[在 GitHub 部署 Hugo 靜態網站](https://chswei.github.io/post/programming/hugo/)

[一个Hugo主题：Hermit](https://ojbk.im/posts/2018/hugo-theme-hermit/)

[Hugo加入留言、觀看人數](https://sunnyday0932.github.io/2020/hugo%E5%8A%A0%E5%85%A5%E7%95%99%E8%A8%80%E8%A7%80%E7%9C%8B%E4%BA%BA%E6%95%B8/)

## 常用語法

```
// 新增文章
hugo new posts/post_name.md
```

```
// 本地端檢視 (http://localhost:1313/)
hugo server
```

```
// 更新網站(新增文章)
hugo
cd public
git add .
git commit -m "update blog"
git push
```

```
// 更新網站(設定)
cd blog
git add .
git commit -m "new feature"
git push
```