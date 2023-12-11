---
author: HKL
categories:
- 默认分类
date: "2023-11-02T22:42:00Z"
slug: install-gentoo-on-existed-linux
status: publish
tags:
- Linux
- Operating
title: Install Gentoo ON Existed Linux without VNC
---


## NO WARRANTY!!! BACKUP ALL YOUR DATA BEFORE OPERATING!
### Operating the following In the Exsiting Linux Distribution

Install Gentoo ON Exsit Linux without VNC

<!--more-->

```
# Download and move the Gentoo stage3 tar package to /tmp (Exsiting Linux Distribution).
root@ali:/tmp/stage3# cp /root/stage3-amd64-systemd-20231029T164701Z.tar.xz .

# Unzip the Gentoo stage3 tar package.
root@ali:/tmp/stage3# tar -xf stage3-amd64-systemd-20231029T164701Z.tar.xz 
root@ali:/tmp/stage3# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  stage3-amd64-systemd-20231029T164701Z.tar.xz  sys  tmp  usr  var
root@ali:/tmp/stage3# rm stage3-amd64-systemd-20231029T164701Z.tar.xz 
root@ali:/tmp/stage3# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  sys  tmp  usr  var
root@ali:/tmp/stage3# cd ..
root@ali:/tmp# ls
# Editing the make file (Actually it can be done later)
root@ali:/tmp/stage3# vim etc/portage/make.conf 
# Editing the repos file (Actually it can be done later)
root@ali:/tmp/stage3# vim etc/portage/repos.conf/gentoo.conf
root@ali:/tmp/stage3# cp /etc/resolv.conf etc/.

# Mount Dynamic links for the system
root@ali:/tmp/stage3# mount --types proc /proc /tmp/stage3/proc
root@ali:/tmp/stage3# mount --rbind /sys /tmp/stage3/sys
root@ali:/tmp/stage3# mount --make-rslave /tmp/stage3/sys
root@ali:/tmp/stage3# mount --rbind /dev /tmp/stage3/dev
root@ali:/tmp/stage3# mount --make-rslave /tmp/stage3/dev
root@ali:/tmp/stage3# mount --bind /run /tmp/stage3/run
root@ali:/tmp/stage3# mount --make-slave /tmp/stage3/run
root@ali:/tmp/stage3# test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
root@ali:/tmp/stage3# mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
root@ali:/tmp/stage3# chmod 1777 /dev/shm /run/shm
root@ali:/tmp/stage3# 


# Now mount to the /tmp Gentoo system
root@ali:/tmp# mount --bind /tmp/stage3/ /tmp/stage3/
root@ali:/tmp# 
root@ali:/tmp# vim /tmp/stage3/^C
root@ali:/tmp# chroot /tmp/stage3/
ali / # 
# export for easily oberve
ali / # export PS1='root@gentoo-chroot-1 #'
root@gentoo-chroot-1 #

```

### Operating the following In 1st chroot environment
```
ali / # export PS1='root@gentoo-chroot-1 # '
root@gentoo-chroot-1 #env-update 
!!! Section 'gentoo' in repos.conf has location attribute set to nonexistent directory: '/var/db/repos/gentoo'
!!! Invalid Repository Location (not a dir): '/var/db/repos/gentoo'
>>> Regenerating /etc/ld.so.cache...
root@gentoo-chroot-1 # mkdir /var/db/repos/gento
root@gentoo-chroot-1 #env-update 

root@gentoo-chroot-1 # lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  61.9M  1 loop 
loop1    7:1    0  63.5M  1 loop 
loop2    7:2    0 111.9M  1 loop 
loop3    7:3    0  79.9M  1 loop 
loop4    7:4    0  40.9M  1 loop 
vda    252:0    0    40G  0 disk 
├─vda1 252:1    0     1M  0 part 
├─vda2 252:2    0   200M  0 part 
└─vda3 252:3    0  39.8G  0 part 

# Mount the hard disk according to your actual environment
root@gentoo-chroot-1 # mount /dev/vda3 /mnt/gentoo
mount: /mnt/gentoo: mount point does not exist.
       dmesg(1) may have more information after failed mount system call.
```

### ! Mount the disk to /mnt/gentoo

Actually, From the step, you can follow the [Gentoo's Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base) to install the System !
 
```
root@gentoo-chroot-1 # mkdir /mnt/gentoo
root@gentoo-chroot-1 # mount /dev/vda3 /mnt/gentoo

root@gentoo-chroot-1 # mount /dev/vda2 /mnt/gentoo/boot/efi
root@gentoo-chroot-1 # lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  61.9M  1 loop 
loop1    7:1    0  63.5M  1 loop 
loop2    7:2    0 111.9M  1 loop 
loop3    7:3    0  79.9M  1 loop 
loop4    7:4    0  40.9M  1 loop 
vda    252:0    0    40G  0 disk 
├─vda1 252:1    0     1M  0 part 
├─vda2 252:2    0   200M  0 part /mnt/gentoo/boot/efi
└─vda3 252:3    0  39.8G  0 part /mnt/gentoo
root@gentoo-chroot-1 #
root@gentoo-chroot-1 # ls /mnt/gentoo/
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var
root@gentoo-chroot-1 # 
root@gentoo-chroot-1 # 
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  stage3-amd64-systemd-20231029T164701Z.tar.xz  sys  tmp  usr  var

root@gentoo-chroot-1 # cd /mnt/gentoo/

# Copy the stage file to /mnt/gentoo (will be the new system's root)

root@gentoo-chroot-1 # cp root/stage3-amd64-systemd-20231029T164701Z.tar.xz  .
root@gentoo-chroot-1 # pwd
/mnt/gentoo
root@gentoo-chroot-1 # ls
bin   dev  home  lib32  libx32      media  opt   root  sbin  srv                                           sys  usr
boot  etc  lib   lib64  lost+found  mnt    proc  run   snap  stage3-amd64-systemd-20231029T164701Z.tar.xz  tmp  var
root@gentoo-chroot-1 # 

root@gentoo-chroot-1 # pwd
/mnt/gentoo
root@gentoo-chroot-1 # 

# Delete the existing linux distribution, NOT ALL DIRs.
root@gentoo-chroot-1 # rm -rf bin home etc lib  lib64 opt root sbin srv usr var
root@gentoo-chroot-1 # ls
boot  dev  lib32  libx32  lost+found  media  mnt  proc  run  snap  stage3-amd64-systemd-20231029T164701Z.tar.xz  sys  tmp
root@gentoo-chroot-1 # rm lib32 libx32
root@gentoo-chroot-1 # ls
boot  dev  lost+found  media  mnt  proc  run  snap  stage3-amd64-systemd-20231029T164701Z.tar.xz  sys  tmp

# Unzip the stage3 as the new system NOW!
root@gentoo-chroot-1 # tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner

root@gentoo-chroot-1 # ls
bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  snap  stage3-amd64-systemd-20231029T164701Z.tar.xz  sys  tmp  usr  var
root@gentoo-chroot-1 # 

# Copy the Configurations or Editing it as you like Now.
root@gentoo-chroot-1 # cp /etc/portage/make.conf /mnt/gentoo/etc/portage/make.conf
root@gentoo-chroot-1 # mkdir --parents /mnt/gentoo/etc/portage/repos.conf
root@gentoo-chroot-1 # cp /etc/portage/repos.conf/gentoo.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
root@gentoo-chroot-1 # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

root@gentoo-chroot-1 # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
root@gentoo-chroot-1 # mount --types proc /proc /mnt/gentoo/proc
root@gentoo-chroot-1 # mount --rbind /sys /mnt/gentoo/sys
root@gentoo-chroot-1 # mount --make-rslave /mnt/gentoo/sys
root@gentoo-chroot-1 # mount --rbind /dev /mnt/gentoo/dev
root@gentoo-chroot-1 # mount --make-rslave /mnt/gentoo/dev
root@gentoo-chroot-1 # mount --bind /run /mnt/gentoo/run
root@gentoo-chroot-1 # mount --make-slave /mnt/gentoo/run
root@gentoo-chroot-1 # test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
root@gentoo-chroot-1 # mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
root@gentoo-chroot-1 # chmod 1777 /dev/shm /run/shm
root@gentoo-chroot-1 # 
```


#### Now CAN chroot to the New Gentoo System  !

```
chroot /mnt/gentoo/ /bin/bash  
source /etc/profile  
ali / # eselect profile list

ali / # emerge -auvDN --with-bdeps=y @world

 * IMPORTANT: 10 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.

Total: 16 packages (3 upgrades, 3 new, 10 reinstalls), Size of downloads: 31022 KiB

Would you like to merge these packages? [Yes/No] Yes

ali / # echo "Hongkong" >/etc/timezone
ali / # emerge --config sys-libs/timezone-data


Configuring pkg...

 * Found a regular file at /etc/localtime.
 * Some software may expect a symlink instead.
 * You may convert it to a symlink by removing the file and running:
 *   emerge --config sys-libs/timezone-data
 * Copying /usr/share/zoneinfo/Hongkong to /etc/localtime.

ali / # echo "en_US.UTF-8 UTF-8 zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
ali / # locale-gen
 * Generating 3 locales (this might take a while) with 2 jobs
 *  (1/3) Generating en_US.UTF-8 ...                                                                                                                        [ ok ]
 *  (2/3) Generating zh_CN.UTF-8 ...                                                                                                                        [ ok ]
 *  (3/3) Generating UTF-8 ...                                                                                                                              [ ok ]
 * Generation complete
 * Adding locales to archive ...                                                                                                                            [ ok ]
ali / # 
ali / # eselect locale list
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   POSIX
  [4]   en_US.utf8
  [5]   zh_CN.utf8
  [6]   C.UTF8 *
  [ ]   (free form)
ali / # es
esac     eselect  
ali / # eselect locale set 4
Setting LANG to en_US.utf8 ...
Run ". /etc/profile" to update the variable in your shell.
ali / # 

ali / # env-update && source /etc/profile
>>> Regenerating /etc/ld.so.cache...
ali / # emerge --ask sys-kernel/gentoo-kernel-bin  
ali / # emerge --depclean
ali / # emerge --ask vim genfstab
ali ~ # genfstab -U /  > /etc/fstab

# Remember to edit the network setting!
ali ~ # vim /etc/systemd/network/20-wired.network

# Install the Grub for booting systems !
ali ~ # emerge --ask --verbose sys-boot/grub
ali ~ # grub-install --target=x86_64-efi --efi-directory=/boot/efi

ali ~ # grub-install --target=x86_64-efi --efi-directory=/boot/efi
Installing for x86_64-efi platform.
Installation finished. No error reported.
ali ~ # grub-mkconfig -o /boot/grub/grub.cf^C
ali ~ # ls /boot/efi/
EFI  NvVars
ali ~ # grub-mkconfig -o /boot/grub/grub.cfg


# Enable DHCP and sshd
ali ~ # systemd-machine-id-setup
ali ~ # systemctl enable systemd-networkd
ali ~ # systemctl enable sshd


ali ~/.ssh # reboot
ali ~/.ssh # Connection closing...Socket close.
```

#### After rebooting the system, expectedly can boot the the New Gentoo System!

```
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
gentoo ~ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:16:3e:01:b8:12 brd ff:ff:ff:ff:ff:ff
    altname enp0s5

gentoo ~ # emerge neofetch

gentoo ~ # neofetch
         -/oyddmdhs+:.                root@gentoo 
     -odNMMMMMMMMNNmhy+-`             ----------- 
   -yNMMMMMMMMMMMNNNmmdhy+-           OS: Gentoo Linux x86_64 
 `omMMMMMMMMMMMMNmdmmmmddhhy/`        Host: Alibaba Cloud ECS pc-i440fx-2.1 
 omMMMMMMMMMMMNhhyyyohmdddhhhdo`      Kernel: 6.1.57-gentoo-dist 
.ydMMMMMMMMMMdhs++so/smdddhhhhdm+`    Uptime: 7 mins 
 oyhdmNMMMMMMMNdyooydmddddhhhhyhNd.   Packages: 324 (emerge) 
  :oyhhdNNMMMMMMMNNNmmdddhhhhhyymMh   Shell: bash 5.1.16 
    .:+sydNMMMMMNNNmmmdddhhhhhhmMmy   Resolution: 1024x768 
       /mMMMMMMNNNmmmdddhhhhhmMNhs:   Terminal: /dev/pts/1 
    `oNMMMMMMMNNNmmmddddhhdmMNhs+`    CPU: Intel Xeon Platinum (2) @ 2.500GHz 
  `sNMMMMMMMMNNNmmmdddddmNMmhs/.      GPU: 00:02.0 Cirrus Logic GD 5446 
 /NMMMMMMMMNNNNmmmdddmNMNdso:`        Memory: 89MiB / 1865MiB 
+MMMMMMMNNNNNmmmmdmNMNdso/-
yMMNNNNNNNmmmmmNNMmhs+/-`                                     
/hMMNNNNNNNNMNdhs++/-`                                        
`/ohdmmddhys+++/:.`
  `-//////:--.

gentoo ~ # 

```

#### Enjoying Gentoo!

refer:

1.[Gentoo's Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base)

2.[ 从现有Linux系统上安装Archlinux ](https://vnf.cc/2020/07/install-archlinux-from-linux/)

3.[Quick Gentoo Setup](https://gtrush.com/2022/06/12/%E6%96%B0%E6%89%8BGentoo%E6%8A%98%E8%85%BE%E8%AE%B0%E5%BD%951-%E5%AE%89%E8%A3%85%E7%AF%87-%E4%BA%8C%E8%BF%9B%E5%88%B6kernel%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/)
