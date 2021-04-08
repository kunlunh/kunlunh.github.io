---
author: HKL
categories:
- 默认分类
date: "2019-12-20T22:33:00Z"
slug: coding-jenkins-jekyll-cos
status: publish
tags:
- Blog
- Operating
- CICD
title: Coding通过Jenkins生成jekyll并发布到腾讯云对象存储Qcloud COS
---

Coding项目通过Jenkins生成jekyll并发布到腾讯云对象存储Qcloud COS

主要是通过Coding Git + 自带的Jenkins构建Docker以及coscmd同步


贴一下Jenkins的实现文件如下：

<!--more-->


```bash
pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]], 
                                                                                                                                                                                                    userRemoteConfigs: [[url: env.GIT_REPO_URL, credentialsId: env.CREDENTIALS_ID]]])
      }
    }
    stage('构建') {
      steps {
        echo '构建中...'
        sh '''docker run --rm \\
  --volume="$PWD:/srv/jekyll" --volume="$PWD/_site:/srv/jekyll/_site" \\
jekyll/builder:$JEKYLL_VERSION \\
  /bin/bash -c "chmod 777 /srv/jekyll && jekyll build --future"'''
        echo '构建完成.'
      }
    }
    stage('部署') {
      steps {
        echo '部署中...'
        sh 'sudo pip install --upgrade pip'
        sh 'sudo pip install -U coscmd'
        sh 'coscmd config -a YOURsecretID -s YOURsecretKEY -b YOURbucket -r ap-guangzhou'
        sh 'coscmd upload -rs --delete ./_site/ /'
        echo '部署完成'
      }
    }
  }
  environment {
    JEKYLL_VERSION = 'latest'
  }
}
```


