---
title: github pages 发布流程
date: 2026-04-30T01:23:03.737Z
draft: false
description: 简单记录从本地编辑博客，推送github pages的流程
tags:
    - github
categories:
    - github
cover:
    image: ""
    alt: ""
    caption: ""
    relative: false
author: 林非
---
## 目标
记录如何推送本地文章到github，特别是在国内，GitHub不稳定的时候

## 步骤
### 一. 配置GitHub pages
流程略写：
    1. 新建仓库，仓库名为 用户名.github.io。
    2. 在settings-pages里，将build and deployment 设置为 GitHub actions
    3. 使用默认的博客程序或者研究Hugo等博客程序进行搭建


### 二.配置git
1. 下载git
2. 复制远程仓库地址：在 GitHub 仓库页面，点击绿色的 “Code” 按钮，然后复制 HTTPS 或 SSH 的仓库地址
3. 在 VS Code 中添加远程仓库：按 Ctrl+Shift+P（macOS 上是 Cmd+Shift+P）打开命令面板，输入并选择 Git: Add Remote
4. 输入名称和地址：
   1. Remote name：通常输入 origin，这是远程仓库的默认别名。
   2. Remote URL：粘贴你刚才从 GitHub 复制的仓库地址。
5. 进行第一次拉取：
   1. 为了避免代码冲突，在推送之前，最好先拉取一下远程仓库的最新代码。按 Ctrl+Shift+P，输入并执行 Git: Pull，将远程仓库内容拉取到本地。
   2. 如果本地分支名和远程分支名不一致（比如本地是 master，远程是 main），需要重命名分支。在 VS Code 的终端里执行命令：git branch -M main。

### 三. 本地博客编写
1. 使用VScode等编辑器，在承载内容的文件夹里，新建md文件，书写博客。
2. 源代码管理-更改行右面的“+”，表示将你做的更改进行暂存
3. 暂存后，在消息里写一下本次变更的内容概况。点上面的“提交”按钮，就推送到git里了
4. 然后再下面的“图形”栏，右侧有几个按钮，找到“推送”图标，点击就可以推送到github上了。

### 四.异常情况处理
1. 如果github当前登陆困难，导致需要连VPN推送。即使你连接了VPN，git默认也不走代理，需要你手工设置一下
   1. 先查看你的VPN的端口，例如"61493"
   2. 可以在左面空白处右键，“在集成终端中打开”，输入两条命令：（最后数字替换为你的VPN端口号）
      1. git config --global http.proxy http://127.0.0.1:61493
      2. git config --global https.proxy http://127.0.0.1:61493
2. 如果返回的消息里，告诉你推送失败了。登陆github中，进到该仓库，点actions，最上面一条应该是红色的，点进去。找到具体的报错。问AI。一般是配置的内容有问题。