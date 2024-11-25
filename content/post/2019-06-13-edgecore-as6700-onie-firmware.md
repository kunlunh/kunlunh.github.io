---
author: HKL
categories:
- 默认分类
cid: 70
date: "2019-06-13T15:38:00Z"
slug: edgecore-as6700-onie-firmware
status: publish
tags:
- Network
- SDN
title: EdgeCore AS6700 ONIE Firmware固件 For HWr01c
updated: 2019/06/28 17:07:21
---


EdgeCore AS6700 ONIE固件，用最新ONIE官方源编译
官方代码有一个bug:

[https://github.com/opencomputeproject/onie/issues/776#issuecomment-501582435](https://github.com/opencomputeproject/onie/issues/776#issuecomment-501582435)


固件如下：硬件版本r01c可用，r01b不可用。


<!--more-->


[https://stu2013jnueducn-my.sharepoint.com/:f:/g/personal/hkl_stu2013_jnu_edu_cn/Ekb3I8k4tIlMoo5lfpuORvgBcILEKELADVYa9qmlq3JgQw?e=l4yngL](https://stu2013jnueducn-my.sharepoint.com/:f:/g/personal/hkl_stu2013_jnu_edu_cn/Ekb3I8k4tIlMoo5lfpuORvgBcILEKELADVYa9qmlq3JgQw?e=l4yngL)

PS:log
```bash
==== Create accton_as6700_32x-r1 u-boot multi-file .itb image ====
Warning (unit_address_vs_reg): Node /images/kernel/hash@1 has a unit name, but no reg property
Warning (unit_address_vs_reg): Node /images/initramfs/hash@1 has a unit name, but no reg property
Warning (unit_address_vs_reg): Node /images/dtb/hash@1 has a unit name, but no reg property
FIT description: ppc kernel, initramfs and FDT blob
Created:         Thu Jun 13 15:25:02 2019
 Image 0 (kernel)
  Description:  accton_as6700_32x-r1 ppc Kernel
  Created:      Thu Jun 13 15:25:02 2019
  Type:         Kernel Image
  Compression:  gzip compressed
  Data Size:    3497115 Bytes = 3415.15 kB = 3.34 MB
  Architecture: PowerPC
  OS:           Linux
  Load Address: 0x00000000
  Entry Point:  0x00000000
  Hash algo:    crc32
  Hash value:   3cb3b42a
 Image 1 (initramfs)
  Description:  initramfs
  Created:      Thu Jun 13 15:25:02 2019
  Type:         RAMDisk Image
  Compression:  uncompressed
  Data Size:    1508544 Bytes = 1473.19 kB = 1.44 MB
  Architecture: PowerPC
  OS:           Linux
  Load Address: 0x00000000
  Entry Point:  0x00000000
  Hash algo:    crc32
  Hash value:   2905a26e
 Image 2 (dtb)
  Description:  accton_as6700_32x-r1.dtb
  Created:      Thu Jun 13 15:25:02 2019
  Type:         Flat Device Tree
  Compression:  uncompressed
  Data Size:    29297 Bytes = 28.61 kB = 0.03 MB
  Architecture: PowerPC
  Hash algo:    crc32
  Hash value:   694c77a8
 Default Configuration: 'accton_as6700_32x'
 Configuration 0 (accton_as6700_32x)
  Description:  accton_as6700_32x-r1
  Kernel:       kernel
  Init Ramdisk: initramfs
  FDT:          dtb
==== Create accton_as6700_32x-r1 ONIE binary image ====
ln -sf /home/hkl/onie/build/images/accton_as6700_32x-r1.itb /home/hkl/onie/build/accton_as6700_32x-r1/onie.itb
==== Create accton_as6700_32x-r1 ONIE updater ====
Building self-extracting ONIE installer image ............. Done.
Success:  ONIE installer image is ready in /home/hkl/onie/build/images/onie-updater-powerpc-accton_as6700_32x-r1:
-rw-r--r-- 1 root root 5224311 Jun 13 15:25 /home/hkl/onie/build/images/onie-updater-powerpc-accton_as6700_32x-r1
Created: /home/hkl/onie/build/images/onie-updater-powerpc-accton_as6700_32x-r1
=== Finished making onie-powerpc-accton_as6700_32x-r1 master-201906131520 ===
```