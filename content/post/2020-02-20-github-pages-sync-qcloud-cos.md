---
author: HKL
categories:
- 默认分类
date: "2020-02-20T12:12:00Z"
slug: github-pages-sync-qcloud-cos
status: publish
tags:
- Blog
- Operating
- CICD
title: Github Pages同步到Qcloud腾讯云对象存储COS
---

以本站为例，配置Github Pages同步到Qcloud腾讯云对象存储COS

主要是由于Github Pages + CloudFalre CDN的方式最近访问经常会出问题，所以不得不考虑将本站在国内也新增一个节点，

很久之前就已经[尝试过](/2019/12/coding-jenkins-jekyll-cos/) 将本站部署到阿里、腾讯、又拍等地方，之前测试过兼容性最好的应该就是又拍云，但是无奈╮(╯▽╰)╭ 腾讯云还有很多优惠券还没使用，所以这次就先将其同步到腾讯云的对象存储+CDN。


方法主要是通过Github的Action功能，

先通过jekyll的docker生成`_pages`文件夹，然后通过`coscmd`同步文件到腾讯的对象存储中。CDN配置比较简单忽略。


主要是贴一下action的实现文件如下：

<!--more-->


`.github/workflows/jekyll.yml`

```bash
{% raw %}
name: Jekyll site CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Build the site in the jekyll/builder container
      run: |
        docker run \
        -v ${{ github.workspace }}:/srv/jekyll -v ${{ github.workspace }}/_site:/srv/jekyll/_site \
        jekyll/builder:latest /bin/bash -c "chmod 777 /srv/jekyll && jekyll build --future"
    - name: Install coscmd
      run: sudo pip install coscmd
      
    - name: Configure coscmd
      env: 
        secret_id: ${{ secrets.SecretId }}
        secret_key: ${{ secrets.SecretKey }}
        bucket: ${{ secrets.BUCKET }}
        region: ap-guangzhou
      run: coscmd config -a $secret_id -s $secret_key -b $bucket -r $region
    - name: Upload to Tencent COS
      run: coscmd upload -rs --delete ./_site/ /
{% endraw %}
```

[.github/workflows/jekyll.yml链接](https://github.com/hiplon/hiplon.github.io/blob/master/.github/workflows/jekyll.yml)

```bash
{% raw %}
secret_id: ${{ secrets.SecretId }}
secret_key: ${{ secrets.SecretKey }}
bucket: ${{ secrets.BUCKET }}
{% endraw %}
```

这块就要在仓库的Settings里面配置好就行。

