---
author: HKL
date: "2013-07-27T16:31:00Z"
slug: kde-gtk-theme
tags:
- Linux
- Opensource
title: kde安装gtk主题(kde-gtk-theme)KDE4下gtk程序美化
---


默认安装的kde桌面使用gtk程序很难看,原因是没装主题 (KDE 4 Theme Integration with GTK Applications)

以下为在Archlinux中KDE美化gtk程序的方法。其它Linux发行版方法雷同。

To better integrate GTK and KDE 4 themes, you can use `QtCurve` and(or) `oxygen-gtk`


首先安装 QtCurve 或 oxygen-gtk

`#pacman -S qtcurve-gtk2 qtcurve-kde4 gtk-kde4`


然后安装  gtk-chtheme

`#pacman -S gtk-chtheme`


打开 gtk-chtheme 更换主题即可

`gtk-chtheme`