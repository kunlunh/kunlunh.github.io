---
author: HKL
categories:
- 默认分类
cid: 47
date: "2017-05-15T19:51:00Z"
slug: fish-in-win10-bash
status: publish
tags:
- Operating
- Windows
title: 在Windows10 Bash中默认启动其他shell
updated: 2019/02/12 14:03:43
---


## **首先，通过apt安装其他shell软件** ##

（以fish为例）

`sudo apt install fish`
![img](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2017/05/bpoo782qct.jpeg)


安装之后使用`fish`命令尝试启动。能成功启动则继续下一步。


<!--more-->


## **设置shell默认启动** ##

由于Windows10 Bash是通过在命令行中`bash`命令直接启动Linux的Bash软件，可通过修改`.bashrc`文件使得fish等shell默认启动。
通过在用户目录`~`中编辑`.bashrc`文件。

`vim .bashrc`
![img](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2017/05/ec8xdeqmp1.jpeg)


并在配置文件首部分加入一下配置信息：
```bash
# Launch fish
if [ -t 1 ]; then
    exec fish
fi
```

![img](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2017/05/ap3xlrim9m.jpeg)



保存文件后推出Bash并重启即可。
![img](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2017/05/1hd34h1nyd.jpeg)

------

选编自：https://www.howtogeek.com/258518/how-to-use-zsh-or-another-shell-in-windows-10/