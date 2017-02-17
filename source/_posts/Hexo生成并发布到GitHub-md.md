---
title: Hexo生成并发布到GitHub.md
date: 2017-02-17 23:46:09
tags: 教程
---

# 安装环境
+ 安装Node.js
+ 安装git

# 安装Hexo
运行命令
`npm install hexo-cli -g`

# Hexo使用

## 生成项目
新建一个blog目录，生成项目
`hexo init`
安装依赖
`npm install`

## 启动项目
本地启动项目
`hexo server`
清除生成文件
`hexo clean`
生成静态文件
`hexo generate`

## 部署到GitHub
首先生成SSH，然后将公钥配置到GitHub上
新建GitHub Pages项目
修改站点配置文件

```yml
deploy:
  type: git
  repo: git@github.com:unlimitedLoop/unlimitedLoop.github.io.git
  branch: master
```

部署
`hexo deploy`

## 在Ubuntu上部署遇到的问题
安装Hexo之后，必须使用sudo运行命令，导致生成的所有文件都需要管理员权限才能更改
解决方法，将node_modules下的文件所有权更改为自己
`sudo chown -R user:group /usr/local/lib/node_modules`
