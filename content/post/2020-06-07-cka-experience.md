---
author: HKL
categories:
- 默认分类
date: "2020-06-07T21:32:00Z"
slug: cka-experience
status: publish
tags:
- Blog
- Operating
title: CKA Exam 2020-06考试经验
---

最近趁打折入了CKA的考试券，从入门到考试学习了一个多星期参加了考试，最后以93%的分数通过考试，分享一下最新的情况。

首先还是说下当前Kubernetes已经基本成为业界的标准，连电信行业也引进了Container Network Function (AKA. CNF) 的做法，所以还是有必要更新一下自己的知识库。

前人已经分享了许多经验了，CKA的考试其实网上也有分享了挺多题库，但是都算比较久，所以仅供参考。我下面只说一些自己考试的一些Bibel。

<!--more-->

考试遇到的题目，这份是比较接近 [https://blog.csdn.net/zhouwenjun0820/article/details/105881669](https://blog.csdn.net/zhouwenjun0820/article/details/105881669)

考试用到的Kubernetes已经是1.18版本，其中一些创建Pod，创建Deployment的命令得要注意，例如kubectl run当前就只能创建Pod，要创建Deployment的话还是要老老实实走kubectl create deployment的命令。

https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete

建议在开始的时候就先配好autocomplete, `alias k=kubectl`, `complete -F __start_kubectl k`, 那么在做题时会节省很多时间

以下的链接均为Kubernetes.io/docs的官网链接，考试的时候可以访问

创建集群和加入集群

```bash
kubeadm init --参数
<记录join信息，或者新建token>
kubeadm join --参数
kubectl apply -f <网络插件>
```

这个链接里的fannel.yml链接反正在考试中很有用，细节就不详说了 https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/

其它的话只要熟练使用--dry-run -oyaml参数，对生成的yaml文件进行一下编辑，再通过`kubectl apply -f <file>`解题，对kubernetes有基本了解的话通过难度都不会太大

而且最新的通过标准好像只要66%，所以估计通过考试问题都不大，所以参加考试的时候不用太紧张了。

国内应该不好下载grc.io上面的镜像创建kubernetes集群, 建议是通过[ubuntu snap microk8s](https://microk8s.io/)部署单节点或者双节点的集群进行练习就可以了。


如有待续。


