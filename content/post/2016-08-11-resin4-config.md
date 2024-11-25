---
author: HKL
categories:
- 默认分类
cid: 32
date: "2016-08-11T11:34:00Z"
slug: resin4-config
status: publish
tags:
- Linux
- Opensource
- Office
title: Resin4配置
updated: 2019/01/29 17:35:22
---


Resin4配置文件发生了较大变化，分为：

app-default.xml  web应用配置
cluster-default.xml  集群配置
resin.xml
resin.properties  会被修改的变量 


一.删除/注释resin社区版本不支持的功能

1.health：
修改resin.xml, 删除

```xml
<resin:import path="${__DIR__}/health.xml"/>  
```


2.LoadBalance
修改resin.xml, 删除

```xml
<web-app id="">  
   <resin:LoadBalance regexp="" cluster="app"/>  
</web-app>  
```


<!--more-->


3.仅需要保留自己使用的cluster，
其它的可以删除
修改resin.xml, 删除cluster id="web"， cluster id="memcached"


二.自定义端口

Resin 运行起来后，一般有这么几个端口

WatchDog 的端口，默认6600
Server 监控端口，默认6800
应用的HTTP端口，默认8080 


1.修改Server端口6800


```xml
<server-multi id-prefix="app-" address-list="${app_servers}" port="6800"/>  
```


2.修改WatchDog端口6600


```xml
<server-multi id-prefix="app-" address-list="127.0.0.1" port="6801">  
    <watchdog-port>6601</watchdog-port>  
</server-multi>
```

3.修改应用端口8080


```xml
<server-multi id-prefix="app-" address-list="127.0.0.1" port="6801">  
    <watchdog-port>6601</watchdog-port>  
    <http address="*" port="8081"/>  
</server-multi>  
```


三.禁用admin/doc/deploy

1.修改resin.properties

```apacheconf
web_admin_enable : false  
session_store : false （每个服务器是一个集群，不需要考虑session 持久化）  
resin_doc : false  
dev_mode:false  
```


2.修改resin.xml
删除

```xml
<host id="" root-directory=".">  
  <!--  
    - webapps can be overridden/extended in the resin.xml  
    -->  
  <web-app id="/" root-directory="webapps/ROOT"/>  
  
  <resin:if test="${resin_doc}">  
    <web-app id="/resin-doc" root-directory="${resin.root}/doc/resin-doc"/>  
  </resin:if>  
</host>  
```


四.添加自定义的应用

1.添加host
修改resin.xml，添加

```xml
<web-app id="/" root-directory="/data/www/cms">  
</web-app>  
```


2.防止避免hash collision dos攻击
form-parameter-max 用来限制每次post submit的参数个数

```xml
<web-app id="/" root-directory="/data/www/cms">  
    <form-parameter-max>100</form-parameter-max>                         
</web-app>  
```


3.日志

```xml
<web-app id="/" root-directory="/data/www/cms">  
    <form-parameter-max>100</form-parameter-max>                         
    <stderr-log path='/data/logs/cms/stderr.log' timestamp='[%Y-%m-%d %H:%M:%S] ' rollover-period='1D'/>  
    <stdout-log path='/data/logs/cms/stdout.log' timestamp='[%Y-%m-%d %H:%M:%S] ' rollover-period='1D'/>  
</web-app>  
```


注意，stdout-log目前只会输出系统中System.out.println()的内容，和以前版本不同。

```xml
<log-handler name="" level="all" path="/data/logs/passport/handler.log"  
	timestamp="[%Y-%m-%d %H:%M:%S]" rollover-period="1D"/>  
```



五.resin集群

1.配置
Resin4支持快速配置cluster，修改resin.properties,将集群的配置依照顺序填进上去即可

```apacheconf
app_servers : 192.168.1.15 192.168.1.16 192.168.1.17  
```

注意：三台机器的配置项需要一致
2.启动

```bash
./bin/resin.sh –conf ./conf/resin.xml start  
```

在启动的时候，有时候会发现启动不成功的情况，可以单台启动，比如：

```bash
./bin/resin.sh –conf ./conf/resin.xml -server app-0 start  
```

其中app-0代表集群中的第一台机器，其他类推
3.部署

```bash
./resin/bin/resinctl deploy /tmp/test.war  
```

部署完,进行启动：

```
    ./resin/bin/resinctl web-app-start test   
```



六.不使用resin集群
修改resin.xml,替换
```
    <server-multi id-prefix="app-" address-list="127.0.0.1" port="6801">  
        <watchdog-port>6601</watchdog-port>  
        <http address="*" port="8081"/>  
    </server-multi>  
```

为
```xml
    <server id="app" address="127.0.0.1" port="6801" >
        <watchdog-port>6601</watchdog-port>
        <http address="*" port="8081"/>
    </server> 
```

