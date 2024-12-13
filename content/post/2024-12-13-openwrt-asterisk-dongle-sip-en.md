---
author: HKL
categories:
- Default Category
date: "2024-12-13T20:42:00Z"
slug: openwrt-asterisk-dongle-gsm-to-sip-en
status: publish
tags:
- Networking
- Operating
title: Converting GSM Calls to SIP with OpenWRT Using a 3G Modem and Asterisk
---

This article demonstrates how to convert GSM calls to SIP calls on an OpenWRT system using a Huawei 3G Modem and the Asterisk suite.

Translated by ChatGPT as the original article is [in Chinese](/2019/12/openwrt-asterisk-dongle-gsm-to-sip/).

### Installing Asterisk16 on OpenWRT

```bash
opkg update
opkg install asterisk16-app-system asterisk16-chan-dongle asterisk16-pjsip asterisk16-codec-ulaw asterisk16-codec-alaw asterisk16-res-rtp-asterisk asterisk16-bridge-simple
```

### Configuring PJSIP as the Default Service and Adding Accounts

Modify `/etc/asterisk/pjsip.conf`:

```bash
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0

[6003]
type=endpoint
transport=transport-udp
context=from-internal
disallow=all
allow=ulaw
auth=6003-auth
aors=6003

[6003-auth]
type=auth
auth_type=userpass
password=6003
username=6003

[6003]
type=aor
max_contacts=1

[6004]
type=endpoint
transport=transport-udp
context=from-internal
disallow=all
allow=ulaw
auth=6004-auth
aors=6004

[6004-auth]
type=auth
auth_type=userpass
password=6004
username=6004

[6004]
type=aor
max_contacts=1
```

### Adding an Internal Dial Plan

Modify `/etc/asterisk/extensions.conf`:

```bash
[from-internal]
exten => _Z.,1,Dial(PJSIP/${EXTEN},60,Trg)
same => n,Hangup()
```

Check extension status during calls:

```bash
asterisk -rvvvv
OpenWrt*CLI> pjsip show contacts
```

### Using IAX for Extensions

For simplified NAT traversal, consider IAX extensions. Install and configure as follows:

```bash
opkg update
opkg install asterisk16-chan-iax2
```

Modify `/etc/asterisk/iax.conf`:

```bash
[general]
bindport=4569
bindaddr=0.0.0.0
iaxcompat=yes
nochecksums=yes
disallow=all
allow=ulaw

[6010]
type=friend
username=6010
secret=6010
context=from-internal
host=dynamic
callerid=6010<6010>
```

Update `/etc/asterisk/extensions.conf`:

```bash
[from-internal]
exten => 6010,1,Dial(IAX2/6010,60,Trg)
```

### Configuring the Dongle Device

Configure `/etc/asterisk/dongle.conf` (adjust details as per your device):

```bash
[general]
interval=20

[defaults]
context=dongle-in
group=0
exten=+862022221234

[dongle0]
imei=123451234512345
```

Check device status:

```bash
asterisk -rvvvv
RiWifi*CLI> dongle show devices
```

### Adding Outgoing and Incoming Call Configurations

Update `/etc/asterisk/extensions.conf`:

```bash
[from-internal]
exten => _1.,1,Dial(Dongle/g0/${EXTEN:1})

[dongle-in]
exten => +862022221234,1,Dial(IAX2/6010,60,Trg)
```

### Testing

#### Incoming Call Test

![Incoming Test 1](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/ppvuess1u8.png)

![Incoming Test 2](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/bu16kf7yx3.jpeg)

#### Outgoing Call Test

![Outgoing Test 1](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/r6wjvf9s6k.png)

![Outgoing Test 2](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/jek9b1pyhn.jpeg)
```

