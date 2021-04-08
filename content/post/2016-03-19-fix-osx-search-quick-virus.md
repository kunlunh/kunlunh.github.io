---
author: HKL
categories:
- 默认分类
cid: 20
date: "2016-03-19T15:11:00Z"
slug: fix-osx-search-quick-virus
status: publish
tags:
- Opensource
- OSX
- Office
title: 解决OSX系统Safari的主页被恶意篡改为search-quick.com的问题
updated: 2019/01/29 16:42:28
---


1.在 `/Library/LaunchDeamons` 找到名如 `com."abc".deamon.plist` 和 `com."abc".helper.plist` 的两个文件并删除掉；
2.在 `/Library/LaunchAgents` 找到名如 `com."abc".agent.plist` 的文件并删除掉；
3.在 `/Library/Application Support` 找到名如 "abc" 的文件夹并删除掉；
4.在 `/System/Library/Frameworks` 里找到 `.framwork` 或者 `vmnet.framwork` 的文件夹并删除掉；
5.重启电脑

其中"abc"是一个随机的字符串，类似一个软件的名字

这个问题是由于安装了不明软件造成的，所以在安装软件时应该格外小心，注意验证软件来源的可靠性。