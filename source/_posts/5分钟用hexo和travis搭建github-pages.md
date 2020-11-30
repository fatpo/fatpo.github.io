---
title: 5分钟用hexo和travis搭建github pages
date: 2020-11-30 22:04:55
tags: 奇技淫巧
---

# 创建一个自己的 git repository 
比如我的用户名是： fatpo
那么创建：
```dtd
https://github.com/fatpo/fatpo.github.io
```

# 安装 hexo
macOS 如果没有npm：
```dtd
brew install node
```
安装 hexo：
```dtd
npm install -g hexo-cli
```

# hexo 生成网站
```dtd
cd /Users/apple/IdeaProjects/
hexo init fatpo.github.io
```
当前目录:
```dtd
AppledeMacBook-Pro% pwd
/Users/apple/IdeaProjects/fatpo.github.io
```
进入 fatpo.github.io 目录：
```dtd
hexo s
```
到时候可以看到网站：
```dtd
http://127.0.0.1:4000
```

# travis-ci
帮助我们省去本地部署`hexo g`+上传 html 文件的工作: 
```dtd
https://travis-ci.com/
```
怎么关联 github 和 travis-ci：
```dtd
https://hexo.io/zh-cn/docs/github-pages
```
自动编排后可以看到网站已经生效了：
```dtd
https://fatpo.github.io/
```