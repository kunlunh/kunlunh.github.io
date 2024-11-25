---
author: HKL
date: "2013-07-27T21:30:00Z"
slug: vidalia-unexpectedly
tags:
- Linux
- Opensource
title: 在Ubuntu使用TOR遇到Vidalia detected that the tor software exited unexpectedly解决方案
---


If you are reading this, its because you have installed Tor and Vidalia in Ubuntu (or Debian) and found out that they do not play nice together.


当你看到这往篇文章的时候。大概是因为你在Ubuntu或者Debian时遇到Vidalia detected that the tor software exited unexpectedly吧。而且英文不太好。

 

You probably got an error:看好了，大概就是这个样子吧！

![alt text][logo]
[logo]: http://m1.img.libdd.com/farm5/1/6308D60663F91957414B4E0924B3FF01_500_453.jpg "Vidalia detected that the tor software exited unexpectedly."


> Vidalia detected that the tor software exited unexpectedly.

> Please check the message log for recent warning or error messages.


<!--more-->


在这之前。在终端机用以下命令关闭TOR ：

```bash
sudo killall tor
```

 <!--more-->


### 第一步: 在终端机输入以下命令 ：



```bash
sudo gedit /etc/tor/torrc
```


#### 接着会在53至60行会出现下面的东西：

```apacheconf
## The port on which Tor will listen for local connections from Tor

## controller applications, as documented in control-spec.txt.

# ControlPort 9051

## If you enable the controlport, be sure to enable one of these

## authentication methods, to prevent attackers from accessing it.

# HashedControlPassword

16:872860B76453A77D60CA2BB8C1A7042072093276A3D701AD684053EC4C

# CookieAuthentication 1
```
 

 

### 第二步:删除 第55行“ #ControlPort 9051”前面的 “#“这个符号:

就是使得 第55行变成下面那样，只是前面没了“#”号

`ControlPort 9051`


之后请看到第58行

`HashedControlPassword`

看看后面的一大串东西

 

 

### 第三步:在终端机输入以下命令:

```bash
tor –hash-password mypassword
```


然后会出现类似的一串字符:

`16:816172DEB125A9CA603A6A8A5C16D0642DA4556E4EC417E6B9AAC9AF0D`


复制这串东西到刚才“看看后面的一大串东西”那里并删除原来的

最后变成这个样子:

```apacheconf
## The port on which Tor will listen for local connections from Tor

## controller applications, as documented in control-spec.txt.

ControlPort 9051

## If you enable the controlport, be sure to enable one of these

## authentication methods, to prevent attackers from accessing it.

#HashedControlPassword

16:816172DEB125A9CA603A6A8A5C16D0642DA4556E4EC417E6B9AAC9AF0D

#CookieAuthentication 1
```
 

 

最后: 保存（当然可以用 ctrl + s） 。并重启TOR :

```bash
sudo /etc/init.d/tor restart
```

 

完成了 打开Vidalia干你想干的事吧.
