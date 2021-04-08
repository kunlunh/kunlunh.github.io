---
author: HKL
categories:
- 默认分类
cid: 28
date: "2016-06-21T14:51:00Z"
slug: autorun-virus
status: publish
tags:
- Coding
- Windows
- Firefox
title: 使用attrib命令解决存储器中毒后文件夹被隐藏的方法
updated: 2019/01/29 17:36:15
---


U盘中了某种文件夹类型的病毒，特别是autorun病毒。杀了毒之后U中的文件夹都被隐藏了。

介绍一个使用attrib命令解决存储器中毒后文件夹被隐藏的方法。
原理是autorun病毒会将原来U盘中的文件和文件夹设置为“系统文件”和“隐藏文件”属性，而一般Windows系统会隐藏“系统文件”。所以只需要使用attrib命令将文件和文件夹设定为初始的一般属性即可。

命令使用方法：

```powershell
attrib   c:\”*” -s -h /s /d
```

其中C为U盘盘符

拓展阅读：attrib命令详解


<!--more-->


attrib命令的作用：显示、设置或删除指派给文件或目录的只读、存档、系统以及隐藏属性。如果在不含参数的情况下使用，则 attrib 命令会显示当前目录中所有文件的属性。其命令使用语法如下：
```powershell
attrib [{+r | -r}] [{+a | -a}] [{+s | -s}] [{+h | -h}]
attrib [[Drive:][Path] FileName] [/s[/d]]
```
参数：
```powershell
+r           设置只读文件属性。
-r            清除只读文件属性。
+a          设置存档属性。
-a           清除存档属性。
+s          设置系统文件属性。
-s           清除系统文件属性。
+h          设置隐藏文件属性。
-h            清除隐藏文件属性。
/s            将 attrib 和任意命令行选项应用到当前目录及其所有子目录中的匹配文件。
/d           将 attrib 和任意命令行选项应用到目录。
/?           在命令提示符下显示帮助。
```
