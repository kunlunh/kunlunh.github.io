---
author: HKL
categories:
- 默认分类
date: "2020-04-13T19:35:00Z"
slug: sms-forward-to-weixin
status: publish
tags:
- Office
- Operating
title: 通过tasker转发短信到微信或者钉钉
---

本文主要实现在Android系统中利用tasker软件以及企业微信应用或者钉钉机器人的Webhook接口将收到的短信方便转发到微信或者钉钉上面，

准备工作，需要在Android手机安装好tasker软件，申请好企业微信号或者钉钉团队，并且有管理员权限

下面主要分三步：

（1）注册企业微信应用或者钉钉机器人，或者接口所需资料

（2）部署tasker任务


（1）注册企业微信应用或者钉钉机器人，或者接口所需资料

企业微信：

企业微信应用需要`企业ID`、`企业应用Secret`、`企业应用AgentId`

<!--more-->

`企业ID` 可以在 首页->我的企业 最下信息找到 https://work.weixin.qq.com/wework_admin/frame#profile

`企业应用Secret` 可以在 首页->应用管理->自建APP 的详情页找到 https://work.weixin.qq.com/wework_admin/frame#apps/modApiApp/

`企业应用AgentId` 可以在 首页->应用管理->自建APP 的详情页找到 https://work.weixin.qq.com/wework_admin/frame#apps/modApiApp/

接下来可以用普通微信扫一扫 首页->我的企业->微信插件 的 `邀请关注` 里面的二维码关注企业微信，那么企业应用的信息就会提醒到普通微信里面

钉钉群聊机器人：

需要先拉三人组一个钉钉群，然后创建聊天机器人，可以再踢掉其他人。 在聊天机器人配置里可以获得Webhook地址。


（2）部署tasker任务

在Android手机Tasker软件里创建 `事件` -> `电话` -> `收到短信` -> `新建任务` -> `新建操作` -> `代码` -> `JavaScriptlet`

然后根据(1)中获取到的信息修改以下脚本并粘贴到`JavaScriptlet`代码内容中：

企业微信：

```javascript
//下面的三个变量值需要修改
var ID = "企业ID";
var SECRET = "企业应用Secret";
var AGENTID = "企业应用AgentId";

//定义post方法
function posthttp(url, data) {
    var xhr = new XMLHttpRequest();
    xhr.addEventListener("readystatechange", function () {
        if (this.readyState === 4) {
            flash(this.responseText); //显示返回消息,可删除本行
        }
    });
    xhr.open("POST", url, false);
    xhr.send(data);
    return xhr.responseText;
}

//定义get方法
function gethttp(url) {
    var xhr = new XMLHttpRequest();
    xhr.addEventListener("readystatechange", function () {
        if (this.readyState === 4) {
            flash(this.responseText); //显示返回消息,可删除本行
        }
    });
    xhr.open("GET", url, false);
    xhr.send();
    return xhr.responseText;
}

//获取token
var gettoken = "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=" + ID + "&corpsecret=" + SECRET;
var ACCESS_TOKEN = JSON.parse(gethttp(gettoken)).access_token;

//发送消息(文本)
var SMSRF = global('SMSRF');
var SMSRB = global('SMSRB');
var SMSRT = global('SMSRT');
var SMSRD = global('SMSRD');
var CONTENT = "发件人: " + SMSRF + "\n时间: " + SMSRT + ",  日期: " + SMSRD + "\n短信内容: " + SMSRB;
var message = JSON.stringify({
    "touser": "@all",
    "msgtype": "text",
    "agentid": AGENTID,
    "text": {
        "content": CONTENT
    },
    "safe": 0
});
var send = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=" + ACCESS_TOKEN;
posthttp(send, message);
```

虽然企业微信可以用markdown的msgtype，但是推送到普通的微信的内容不支持markdown显示，所以建议还是用text的msgtype方便显示。


钉钉：

```javascript
///修改为机器人webhook链接最后的token信息
var ACCESS_TOKEN = "Your Access Token";

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
var CONTENT = "发件人: " + SMSRF + "\n时间: " + SMSRT + ", 日期: " + SMSRD + "\n > 短信内容: " + SMSRB;
var message = JSON.stringify({"msgtype": "markdown", 
"markdown": { "title": TITLE, "text": CONTENT } });
var send = "https://oapi.dingtalk.com/robot/send?access_token="; + ACCESS_TOKEN;
posthttp(send, message);
```
