---
author: HKL
categories:
- 默认分类
date: "2019-11-21T22:13:00Z"
slug: elk-h3c-huawei-log
status: publish
tags:
- Networking
- Operating
title: ELK收集处理Huawei/H3C交换机日志
---

通过ELK logstash模块收集处理交换机日志并进行可视化处理

这里主要是写一下logstash grok的对交换机日志的正则实现，之后再补充可视化的分析模板

1.Huawei Switch 华为交换机

效果如下

<!--more-->

![Huawei Switch ELK][1]

logstash配置文件如下：

`/etc/logstash/conf.d/hw-log.conf`
```bash
input {
   udp {
    port => 2514
    type => syslog 
   }
}
filter {
 if [type] == "syslog" {
 	grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>(?<switchtime>.*) %{DATA:hostname} %{DATA:ddModuleName}/%{POSINT:severity}/%{DATA:Brief}:%{GREEDYDATA:message}"
            }
            remove_field => [ "timestamp" ]
            add_field => {
                "severity_code" => "%{severity}"
            }
            overwrite => ["message"]
        }
  }
  mutate {
        gsub => [
            "severity", "0", "Emergency",
            "severity", "1", "Alert",
            "severity", "2", "Critical",
            "severity", "3", "Error",
            "severity", "4", "Warning",
            "severity", "5", "Notice",
            "severity", "6", "Informational",
            "severity", "7", "Debug"
        ]
    }
}
output {
  elasticsearch {
     hosts => ["127.0.0.1:9200"]
     index => "switchlog"
  }
}

```

2.H3C Switch 华三交换机

效果如下
![H3C Switch ELK][2]

logstash配置文件如下：

`/etc/logstash/conf.d/h3c-log.conf`
```bash
input {
   udp {
    port => 3514
    type => syslog 
   }
}
filter {
 if [type] == "syslog" {
    grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>%{DATA:hostname} %%%{DATA:vvmodule}/%{POSINT:severity}/%{DATA:digest}: %{GREEDYDATA:message}"
            }
            add_field => {
                "severity_code" => "%{severity}"
            }
            overwrite => ["message"]
        }     
 }

  mutate {
    gsub => [
            "severity", "0", "Emergency",
            "severity", "1", "Alert",
            "severity", "2", "Critical",
            "severity", "3", "Error",
            "severity", "4", "Warning",
            "severity", "5", "Notice",
            "severity", "6", "Informational",
            "severity", "7", "Debug"
        ]  
  }
}
output {
  elasticsearch {
     hosts => ["127.0.0.1:9200"]
     index => "h3clog-logstash"
     template_overwrite => true
  }

```

  [1]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/gs9rnjtshh.png
  [2]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/o4fgoj6hny.png