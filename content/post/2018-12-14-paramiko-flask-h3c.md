---
author: HKL
categories:
- 默认分类
cid: 62
date: "2018-12-14T12:47:00Z"
slug: paramiko-flask-h3c
status: publish
tags:
- Coding
- Network
- Operating
title: 使用python的paramiko加flask模块实现H3C设备实时ssh信息查询
updated: 2019/02/12 14:03:07
---


1.本文主要初步实现一个通过Web实时查询H3C网络设备的终端MAC信息所在端口查询，这个是通过实际网络环境设计的操作逻辑，因而代码部分仅供参考
   
2.系统架构

<!--more-->

![The System Arch](https://img.jnuer.com/img/2018/12/20181214141225.png "The System Arch")

主要是通过flask实现了一个Web界面，通过ajax调用后台接口，后台接口通过paramiko ssh模块在交换机上执行ssh命令，将结果处理后返回给前端Web的一个过程。

3.The code

(1)flask部分
```python
from flask import Flask
from flask import render_template
from flask import request,jsonify
import sshsearch
from sshsearch import *

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/testAjax', methods=['GET', 'POST'])
def testAjax():
	building = request.form['building']
	mac = request.form['mac']
	if building == 'A1' :
		hostname = '10.1.1.1'
		tag = 'v5'
	elif building == 'A2' :
		hostname = '10.1.1.2'
		tag = 'v5'
	elif building == 'A3' :
		hostname = '10.1.1.3'
		tag = 'v5'
	elif building == 'A4' :
		hostname = '10.1.1.4'
	elif building == 'B1' :
		hostname = '172.16.0.1'
		tag = 'v5'
	elif building == 'B2' :
		hostname = '172.16.0.2'
		tag = 'v7'
	res = swsearch(hostname,mac,tag)
	
	return jsonify(result = res)


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8010)
```

(2)paramiko部分
```python
def swsearch(hostname,mac,tag) :
	tag = tag
	result_dic = {}
	if tag == 'v7' :
		hostname = hostname
		username = 'netadmin'
		password = 'Net!@#'
		paramiko.util.log_to_file('syslogin.log')     #发送paramiko日志到syslogin.log文件
		searchcmd = 'dis mac-address | include ' + mac	
		ssh = paramiko.SSHClient()          #创建一个SSH客户端client对象
		ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
		ssh.connect(hostname=hostname,username=username,password=password,allow_agent=False,look_for_keys=False)    #连接汇聚交换机
		stdin,stdout,stderr = ssh.exec_command(searchcmd)      #调用远程执行命令方法exec_command()
		result1 = stdout.readlines()
		ssh.close()
		if len(result1) <= 8 :
			result_dic[1] = 'Not Found'
		else :
			i = 0
			j = 1
		for line in result1 :
			line = line.strip('\n')
			if i < 8 :
				i = i + 1
			else :
				tecmd = '\ndis lldp nei int G1/0/' + line[49:51] + ' verbose | include "Management address                :"\n'   #组成命令查看接入交换机IP地址的命令
				ssh = paramiko.SSHClient()
				ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
				ssh.connect(hostname=hostname,username=username,password=password,allow_agent=False,look_for_keys=False)   #连接汇聚交换机
				stdin,stdout,stderr = ssh.exec_command(tecmd)
				out1 = stdout.read()
				loc = out1.find(': ')  #在结果中定位交换机IP位置
				ip = out1[loc+2:loc+20].strip('\n')   #得到接入交换机IP
				ssh.close()
				ssh = paramiko.SSHClient()
				ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
				ssh.connect(hostname=ip,username=username,password=password,allow_agent=False,look_for_keys=False)  #连接接入交换机
				seccmd = 'dis mac-address | include ' + line[0:14]
				stdin,stdout,stderr = ssh.exec_command(seccmd)
				out2 = stdout.read()
				loc2 = out2.find('Eth')  #定位接入交换机结果所在行
				ssh.close()
				result2 = out2[loc2-8:loc2+15] #粗略定位接入交换机端口
				all_result = 'MAC: ' + line[0:14] + '\n' + 'Access Switch IP: ' + ip + '\n' + 'Port: ' + result2 + '\n' + 'VLAN: ' + line[17:21] + '\n' + 'BAGG: ' + line[49:51] + '\n'
				result_dic[j] = all_result 
				j = j + 1
				
	elif tag == 'v5' :
	......
```
(3)具体代码已经放在github上面
[https://github.com/hiplon/h3c-search](https://github.com/hiplon/h3c-search)

4.总结
H3C设备操作系统有comware v5和comware v7两种，这两个系统虽然大体使用起来感觉差不多，但是具体到字符的返回以及操作指令的细节处还是有一些区别，在这次实现功能过程不得不打tag区分操作系统进行具体的处理。

所以国内的网络设置系统细节还是有待提高。
