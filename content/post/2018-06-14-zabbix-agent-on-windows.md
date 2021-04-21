---
author: HKL
categories:
- 默认分类
cid: 51
date: "2018-06-14T23:11:00Z"
slug: zabbix-agent-on-windows
status: publish
tags:
- Operating
- Linux
- Windows
title: 安装zabbix的windows系统agent
updated: 2019/02/12 14:02:37
---


一、下载软件

首先从官网下载agent软件，解压
https://www.zabbix.com/downloads/3.4.6/zabbix_agents_3.4.6.win.zip

二、编辑配置文件
修改配置文件`zabbix_agentd.win.conf` 主要修改以下的选项


<!--more-->

```ini
EnableRemoteCommands=1
LogRemoteCommands=1
Server=Your_Zabbix_Server_IP_Address
Hostname=The_Client_Name
```

三、安装以及启动zabbix agent
用管理员权限打开命令行工具

用以下命令安装以及启动zabbix agent

```powershell
PATH_to_Zabbix_Agent\zabbix_agentd.exe --config PATH_to_Zabbix_Conf\zabbix_agentd.win.conf --install
    
PATH_to_Zabbix_Agent\zabbix_agentd.exe --config PATH_to_Zabbix_Conf\zabbix_agentd.win.conf --start
```

PS:关闭以及卸载zabbix agent命令为：

关闭：

```powershell
PATH_to_Zabbix_Agent\zabbix_agentd.exe --stop
```

卸载:

```powershell
PATH_to_Zabbix_Agent\zabbix_agentd.exe --config
PATH_to_Zabbix_Conf\zabbix_agentd.win.conf --uninstall
```

四、开放防火墙的相关监控端口

默认监控端口为tcp/10050

五、在zabbix server添加该监控Host

1、添加主机， 

![IMG1][1]

2、Host name要与zabbix_agentd.win.conf的Hostname一致，

![IMG2][2]

3、添加windwos的templates，
 
![IMG3][3]

4、若Host列表中的ZBX按钮显示绿色则代表zabbix server已成功与监控Host连接成功。 

![IMG4][4]


  [1]: http://img.jnuer.com/img/2018/06/01.jpg
  [2]: http://img.jnuer.com/img/2018/06/02.jpg
  [3]: http://img.jnuer.com/img/2018/06/03.jpg
  [4]: http://img.jnuer.com/img/2018/06/04.jpg