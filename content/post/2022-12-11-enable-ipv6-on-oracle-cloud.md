---
author: HKL
categories:
- 默认分类
date: "2022-12-11T16:22:00+08:00"
slug: enable-ipv6-on-oracle-cloud
status: publish
tags:
- Linux
- Operating
title: Enable IPv6 on Oracle Cloud Infrastructure
---

In this post, Talking about how to enable IPv6 on Oracle Cloud environment and how to assign IPV6 address to your VPS.

<!--more-->



**（1）Enable IPv6:**

1. Log into your Oracle Cloud account. Choose networking -> Virtual Cloud Networks 

2. Find out your existing VCN (Virtal Cloud Networks), Click it.

3. You should land on VCN’s Subnets page as show below:

![PIC1][1]

4. Change to CIDR Blocks page , then click Add IPv6 CIDR Block button to add a new IPv6 block in. 

![PIC2][2]

You will get a pop up window to confirm you want to enable ipv6. Click Confirm to continue.

5. After you confirmed to enable ipv6 support, a new ipv6 segment (/56 block) will be assigned to you.

![PIC3][3]

**（2）Create IPv6 Subnet:**

1.Click the existing subnet for Resources panel’s Subnets page:

![PIC4][4]

2. Click edit button then check “Enable IPv6 CIDR Block”

![PIC5][5]

3. Enter a new HEX character between 00-FF to assign a /64 subnet from a block /56. 

**（3）Create Security Rules for Ingress and Egress IPv6 Traffic:**

Ingress rule for all IPv6 Traffic:

![PIC6][6]

Egress rule for all IPv6 Traffic:

![PIC7][7]

**（4）Default IPv6 Route:**

In your Route Rules, there is already one IPv4 default route in place. 

We will also need to add a default IPv6 route in. The option is same concept as your ipv4 default route.

Since it is for all ipv6 traffic, destination CIDR block is ::/0. 

![PIC8][8]

**（5）Assign An IPv6 Address to your instance:**

Go to you instance’s configuration page, which you will find Resources panel at the left of page.

Click Attached VNICs, then choose existing VNIC to click. 

![PIC9][9]

You can assign a new ipv6 or randomly let OCI assign one for you without entering anything, just click Assign button:

![PIC10][10]

After a couple of seconds, one IPv6 address will be assigned to your VNIC. 

**（6）Acquire This IPv6 Address From Your VPS:**

![PIC11][11]


REF:

https://www.51sec.org/2021/09/20/enable-ipv6-on-oracle-cloud-infrastructure/

[1]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/1.png
[2]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/2.png
[3]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/3.png
[4]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/4.webp
[5]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/5.png
[6]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/6.png
[7]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/7.png
[8]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/8.png
[9]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/9.png
[10]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/10.png
[11]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2022/12/11.png