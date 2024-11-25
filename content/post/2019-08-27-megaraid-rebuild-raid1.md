---
author: HKL
categories:
- 默认分类
date: "2019-08-27T07:11:00Z"
slug: megaraid-rebuild-raid1
status: publish
tags:
- Hardware
- Operating
title: megacli修复raid1硬盘
---

使用megaraid修复raid1掉线硬盘


使用说明：

<!--more-->

查看硬盘状态
```bash
~# megacli -PDList -aAll -NoLog | grep 'Firmware state'
Firmware state: Unconfigured(bad)
Firmware state: Online, Spun Up

```

将`Unconfigured(bad)`调整为可用的`good`状态

```bash
~# megacli  -PDMakeGood -PhysDrv[252:0] -a0
~# megacli -PDList -aAll -NoLog | grep 'Firmware state'
Firmware state: Unconfigured(good), Spun Up
Firmware state: Online, Spun Up
```


检查掉线硬盘

```bash
~# megacli -pdgetmissing -a0
                                     
    Adapter 0 - Missing Physical drives

    No.   Array   Row   Size Expected
    0     0       1     952720 MB

```

重新导入raid配置，掉线硬盘进入Rebuild

```bash
~# megacli  -CfgForeign -Import -a0
~# megacli -PDList -aAll -NoLog | grep 'Firmware state'
Firmware state: Rebuild
Firmware state: Online, Spun Up

~# megacli -PDRbld -ShowProg -PhysDrv[252:0] -aAll
                                     
Rebuild Progress on Device at Enclosure 252, Slot 0 Completed 4% in 4 Minutes.

Exit Code: 0x00
```

Rebuild后硬盘恢复在线
```bash
~# megacli  -PDList -a0 |grep "Firmware state"
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up

```
