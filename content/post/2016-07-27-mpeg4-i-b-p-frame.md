---
author: HKL
categories:
- 默认分类
cid: 29
date: "2016-07-27T09:37:00Z"
slug: mpeg4-i-b-p-frame
status: publish
tags:
- Coding
- ffmpeg
- Music
title: MPEG4视频中，I帧、p帧、B帧的判定（转载）
updated: 2019/01/29 17:35:59
---


mpeg4的每一帧开头是固定的：00 00 01 b6，那么我们如何判断当前帧属于什么帧呢？在接下来的2bit，将会告诉我们答案。注意：是2bit，不是byte，下面是各类型帧与2bit的对应关系：

```ini
　　00: I Frame

　　01: P Frame

　　10: B Frame　
```

为了更好地说明，我们举几个例子，以下是16进制显示的视频编码：


<!--more-->


```ini
　　00 00 01 b6 10 34 78 97 09 87 06 57 87 ……                             I帧

　　00 00 01 b6 50 78 34 20 cc 66 b3 89 ……                                  P帧

　　00 00 01 b6 96 88 99 06 54 34 78 90 98 ……                              B帧
```
下面我们来分析一下为什么他们分别是I、P、B帧
```ini
　　0x10 = 0001 0000

　　0x50 = 0101 0000

　　0x96 = 1001 0100　
```
大家看红色的2bit，再对照开头说的帧与2bit的对应关系，是不是符合了呢？

下面给出一段c++代码供大家参考：
```c

switch(buf[i] & (BYTE)0xc0)
{
case 0x00:
    //I Frame
    break;
case 0x40:
    //P Frame
    break;
case 0x80:
    //B Frame
    break;
default:
    break;
}
```


From:
http://www.360doc.com/content/11/0718/17/474846_134326279.shtml

