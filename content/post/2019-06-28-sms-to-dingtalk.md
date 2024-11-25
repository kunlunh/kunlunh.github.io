---
author: HKL
categories:
- 默认分类
cid: 72
date: "2019-06-28T17:06:00Z"
slug: sms-to-dingtalk
status: publish
tags:
- Operating
title: tasker调用钉钉机器人实现短信转钉钉
updated: 2019/06/28 17:41:06
---


tasker可以通过javascriptlet方法调用钉钉群聊机器人api接口可以实现短信转钉钉

tasker配置可以参考以下文章

https://ishare.cf/2019/04/06/forwarded-sms-to-dingtalk/

我与文章不同处主要是：

1.使用“群聊机器人”而不是“企业内部应用”；

2.修正了群聊机器人与企业内部应用脚本的不同之处；


过程主要不同就是申请群聊机器人并获得其API access token

配置脚本的改动，其它过程可完全参照以上文章



群聊机器人配置如下：


<!--more-->


```javascript
var ACCESS_TOKEN = "your_access_token";

//定义post方法
function posthttp(url, data) {
    var xhr = new XMLHttpRequest();
    xhr.addEventListener("readystatechange", function () {
        if (this.readyState === 4) {
            flash(this.responseText); //显示返回消息,可删除本行
        }
    });
  
    xhr.open("POST", url, false);
  xhr.setRequestHeader("Content-type","application/json");
    xhr.send(data);
    return xhr.responseText;
}


//发送消息(文本)
var SMSRF = global('SMSRF');
var SMSRB = global('SMSRB');
var SMSRT = global('SMSRT');
var SMSRD = global('SMSRD');
var TITLE = "Message From " + SMSRF;
var CONTENT = "发件人: " + SMSRF + "\n时间: " + SMSRT + ",  日期: " + SMSRD + "\n短信内容: " + SMSRB;
var message = JSON.stringify({"msgtype": "markdown", 
"markdown": { "title": TITLE, "text": CONTENT  } });
var send = "https://oapi.dingtalk.com/robot/send?access_token=" + ACCESS_TOKEN;
posthttp(send, message);
```