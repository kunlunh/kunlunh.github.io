---
author: HKL
categories:
- 默认分类
date: "2019-09-01T20:21:00Z"
slug: esight-to-dingtalk-robot
status: publish
tags:
- Networking
- Operating
title: Huawei esight to 钉钉dingding (RESTful API)
---

Huawei esight告警本身不能使用dingtalk,wechat等webhook api，但是其自带了一个HTTPS SMS Server，经过分析，可以通过这个功能将其转换成其它API接口可用的数据。


![package](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/09/kfa7qb7vmx.png)

抓包看了一下，这明显是个Get方法，Huawei esight直接当成了post写，也是666，所以正常情况我们不是要在esight的HTTPS SMS Server将方法改成GET

![att](https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/09/lo8t8we2zm.png)

实现很简单，我就是用了flask处理了一下拿到的数据，然后再重组一下发到新的API接口就行了。

简单地实现代码如下：

<!--more-->

查看硬盘状态
```python
from flask import Flask, request, json
import requests

def senddatatodingtalk(alert_message):
    postdata=alert_message
    url='https://oapi.dingtalk.com/robot/send?access_token=DINGTALK_ROBOT_TOKEN'
    message_send={
	"msgtype": "text",
	"text": {"content": postdata},
	}
    headers={'Content-Type': 'application/json'}
    fb=requests.post(url,data=json.dumps(message_send),headers=headers)
    return fb

app = Flask(__name__)
 
@app.route('/')
def hello_world():
    return 'smsrevice'
 
@app.route('/smstodingtalk', methods=['GET','POST'])
def smstodingtalk():
    username = request.args.get('username')
    if username == 'artisan':
    	message = request.args.get('content')
    	print senddatatodingtalk(message)
    	return 'Success'
    else:
	return 'Error'
 
if __name__ == '__main__':
    app.run(host='0.0.0.0',port=8080,debug=0)
```

[https://github.com/hiplon/esight2dingtalk](https://github.com/hiplon/esight2dingtalk)
