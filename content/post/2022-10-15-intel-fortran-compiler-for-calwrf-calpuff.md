---
author: HKL
categories:
- 默认分类
date: "2022-10-15T08:48:00+08:00"
slug: intel-fortran-compiler-for-calwrf-calpuff
status: publish
tags:
- Linux
- Operating
title: Ubuntu 22.04 LTS 基于 Intel Fortran Compiler 2022.2.0 Release编译CALPUFF CALMET CALWRF
---

本文主要介绍在Ubuntu 22.04 LTS 系统基于 Intel Fortran Compiler 2022.2.0 Release 编译 CALPUFF CALMET CALWRF。


<!--more-->


**（0）CALPUFF CALMET CALWRF 源码的下载:**

CALMET: http://src.com/calpuff/download/Mod7_Files/CALMET_v6.5.0_L150223.zip
CALPUFF: http://src.com/calpuff/download/Mod7_Files/CALPUFF_V7.2.1_L150618.zip
CALWRT: http://src.com/calpuff/download/Mod7_Files/CALWRF_v2.0.3_L190426.zip


解压各源代码文件并对文件大小写进行转换：

```bash
for fname in *; do mv $fname `echo $fname|tr [A-Z] [a-z]`; done
```


**（1）安装Intel Fortran Complier:**

下载Intel Fortran Complier: https://registrationcenter-download.intel.com/akdlm/irc_nas/18909/l_fortran-compiler_p_2022.2.0.8773_offline.sh

```bash
./intel_fortran-compiler_p_2022.2.0.8773_offline.sh
```

然后提示Step by Step 操作安装

安装完之后：
在`/home/username/intel/oneapi`文件夹

编译器在：
`/home/username/intel/oneapi/compiler/2022.2.0/linux/bin/ifx`

创建软链接

```bash
sudo ln -s /home/username/intel/oneapi/compiler/2022.2.0/linux/bin/ifx /usr/bin/ifx
```

然后加载相关lib：

```bash
export LD_LIBRARY_PATH="/home/username/intel/oneapi/compiler/2022.2.0/linux/compiler/lib/intel64_lin"
```

**（2）对各个组件进行编译:**

CALPUFF:

```bash
username@usernamemodelcal:/data/calpuff/v7/CALPUFF_v7.2.1_L150618$ /home/username/intel/oneapi/compiler/2022.2.0/linux/bin/ifx modules.for calpuff.for -o /home/username/calpuff
```

CALMET:

```bash
/home/username/intel/oneapi/compiler/2022.2.0/linux/bin/ifx calmet.for -o calmet
```

CALWRF:

安装 netcdf-c库

```bash
sudo apt install libnetcdf-dev libnetcdff7
```

安装netcdf-fortran
下载：https://downloads.unidata.ucar.edu/netcdf-fortran/4.6.0/netcdf-fortran-4.6.0.zip


需要修改configure文件

```bash
vim
:%s/ifort/ifx/g
```

执行编译

```bash
./configure

make 

sudo make install 
```

然后进行CALWRT编译:

```bash
ifx  -o calwrf -I/usr/include/ calwrf.f -L/usr/lib/ -lnetcdf -lnetcdff 
```

做链接方便执行:

```bash
username@usernamemodelcal:~$ sudo ln -s /data/calpuff/v7/CALWRF_v2.0.3_L190426/code/calwrf /usr/bin/calwrf
username@usernamemodelcal:~$ sudo ln -s /data/calpuff/v7/CALMET_v6.5.0_L150223/calmet /usr/bin/calmet
username@usernamemodelcal:~$ sudo ln -s /data/calpuff/v7/CALPUFF_v7.2.1_L150618/calpuff /usr/bin/calpuff
```


**（3）测试:**

使用编译后的calwrf对WRF文件进行计算

```bash
username@usernamemodelcal:~/testfile$ cat calwrf.inp 
Create 3D.DAT file for WRF output
calwrf.lst          ! Log file name
test5.m3d ! Output file name
1,10,1,10,1,10    ! Beg/End I/J/K ("-" for all)
2022083100          ! Start datetime (UTC yyyymmddhh, "-" for all)
2022090123          ! End   datetime (UTC yyyymmddhh, "-" for all)
2                   ! Number of WRF output files
/data/wrfout/wrfout_d03_2022-08-31_00:00:00 ! File name of wrf output (Loop over files
/data/wrfout/wrfout_d03_2022-09-01_00:00:00 ! File name of wrf output (Loop over files


username@usernamemodelcal:~/testfile$ calwrf calwrf.inp 
  Control inp file:calwrf.inp
  2D.DAT flag input not exist, set to one
  Set 2D.DAT flag to 1:           1
  Default 2D.DAT filename:test5.m2d
  Open WRF netcdf file            1 : 
 /data/wrfout/wrfout_d03_2022-08-31_00:00:00
  N_TIMES:          24
  
 Processing GLOBAL ATTRIBUTES:
  
 Warning: Attribute not exist:    3  DYN_OPT         -43
 Check whether this att. is critical
 Process:    1  2022-08-31_00:00:00    2022083100
 Process:    2  2022-08-31_01:00:00    2022083101
 Process:    3  2022-08-31_02:00:00    2022083102
 Process:    4  2022-08-31_03:00:00    2022083103
 Process:    5  2022-08-31_04:00:00    2022083104
 Process:    6  2022-08-31_05:00:00    2022083105
 Process:    7  2022-08-31_06:00:00    2022083106
 Process:    8  2022-08-31_07:00:00    2022083107
 Process:    9  2022-08-31_08:00:00    2022083108
 Process:   10  2022-08-31_09:00:00    2022083109
 Process:   11  2022-08-31_10:00:00    2022083110
 Process:   12  2022-08-31_11:00:00    2022083111
 Process:   13  2022-08-31_12:00:00    2022083112
 Process:   14  2022-08-31_13:00:00    2022083113
 Process:   15  2022-08-31_14:00:00    2022083114
```