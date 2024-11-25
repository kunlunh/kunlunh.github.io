---
author: HKL
categories:
- 默认分类
cid: 69
date: "2019-04-16T09:04:36Z"
slug: sddm-permit-root-login
status: publish
tags:
- Linux
title: sddm启动root登陆kde
updated: 2019/04/16 09:04:36
---


1.`cp /usr/share/doc/sddm/sddm.conf /etc/sddm.conf`

2.`vim /etc/sddm.conf`

```bash
[Users]
# Default $PATH for logged in users 
DefaultPath=/bin:/usr/bin

# Comma-separated list of shells.
# Users with these shells as their default won't be listed
HideShells=/usr/sbin/nologin,/bin/false

# Comma-separated list of users that should not be listed
HideUsers=sync
#通过HideShells和HideUsers隐藏应用用户

# Maximum user id for displayed users
MaximumUid=60000

# Minimum user id for displayed users
MinimumUid=0
#主要是这一行MinimumUid修改为0后才能使用root用户

# Remember the session of the last successfully logged in user
RememberLastSession=true

# Remember the last successfully logged in user
RememberLastUser=true
```

3.`vim /etc/pam.d/sddm`
```bash
#auth    required        pam_succeed_if.so user != root quiet_success
auth    sufficient      pam_succeed_if.so user ingroup nopasswdlogin
```
