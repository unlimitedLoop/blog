---
title: idea
date: 2017-02-17 15:55:15
tags: 日志

---

# 好用的插件

- Acejump，在可视区域跳转
- Check-style，FindBugs，代码风格检查以及静态分析器
- Grep console，根据控制台日志级别区分不同显示
- Gsonformat，将json解析成实体类
- IdeaVim，vim插件
- JProfile，Java代码分析器
- JRebel，热加载
- LiveEdit，静态页面实时预览
- Lombok
- Markdown support
- Maven Helper，maven分析器
- Mongo plugin，mongo客户端
- PlantUML
- RegexpTester
- Save Actions，保存文件的时候触发代码格式化以及优化引入
- String Manipulation，字符串处理
- JVM Debugger Memory View

## Linux下无法使用Git插件更新代码的问题

在`git`设置`SSH execuable`选择Native

## 快捷键

- `ctrl+w`选择一个单词
- `ctrl+shift+alt+f7`查找使用范围设置
- `f12`重新打开关闭的tool window
- `alt+shift+insert`进入块选择模式
- `Back-Forward`跳转
- `Basic-SmartType`补全
- `Run context configuration`运行当前文件
- `Debug context configuration`Debug当前文件
- `Rebuid`Debug时更新改动的文件
- `Goto Next Splitter`分窗口时相互跳转

### 设置可以打开的最大Tab数目

搜索`Tab limit`设置相应数目

## 使用Fira Code字体

IDEA安装的时候自带了这个字体，但是系统其他编辑器无法使用，首先进入windows的字体文件夹，删除所有的Fira字体，然后在重新安装下载的Fira字体

Windows系统下，如果另外安装了Fira字体，在设置中`Appearance-Antialiasing-Editor-Greyscale`才能正常渲染字体

## 配置使用intellij-jdk

使用IDEA的特殊字体，将多个符号连接起来，会有性能问题，需要使用intellij-jdk来运行IDEA

下载[intellij-jdk](https://bintray.com/jetbrains/intellij-jdk/openjdk8-windows-x64#files/)

配置环境变量

- 32位配置IDEA_JDK
- 64位配置IDEA_JDK_64

执行下载的jdk根目录

:smiley:
