---
author: HKL
categories:
- 默认分类
date: "2024-11-25T13:21:00+08:00"
slug: github-action-to-upyun-alioss
status: publish
tags:
- Devops
- Operating
title: Github Action to Upyun and Aliyun OSS
---

There are some steps to upload files to Aliyun OSS and Upyun OSS using Github Action.

This article is mainly about examples how to upload files to Aliyun OSS and Upyun OSS using Github Action.

Just for reference, please modify the content according to your needs.

<!--more-->

## Examples for Upyun with hugo (this blog):

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to UPyun

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    environment: upyunkey
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --minify
  # Deployment job
      - name: deploy-to-upyun
        env:
          UPX_SERVICENAME: ${{ secrets.UPX_SERVICENAME }}
          UPX_OPERATOR: ${{ secrets.UPX_OPERATOR }}
          UPX_PASSWORD: ${{ secrets.UPX_PASSWORD }}
          LOCAL_DIR: ./public/
          REMOTE_DIR: /
        run: |
          wget https://github.com/upyun/upx/releases/download/v0.4.8/upx_0.4.8_linux_amd64.tar.gz || { echo "Download failed"; exit 1; }
          tar -xvf upx_0.4.8_linux_amd64.tar.gz || { echo "Extracting failed"; exit 1; }
          chmod +x ./upx || { echo "Permission setting failed"; exit 1; }
          echo "${UPX_OPERATOR} is operating"
          ./upx login ${UPX_SERVICENAME} ${UPX_OPERATOR} ${UPX_PASSWORD}
          ./upx sync --delete ${LOCAL_DIR} ${REMOTE_DIR} -v
          echo 'Finished.'
```

## Examples for Aliyun OSS (gzcs.github.io):

```yaml
name: OSS Deploy Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

env:
  TZ: Asia/Shanghai

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: OSSDEV
    steps:
      - name: Git Configuration
        run: |
          git config --global core.quotePath false
          git config --global core.autocrlf false
          git config --global core.safecrlf true
          git config --global core.ignorecase false
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
          
      - name: Build Pages
        run: |
          mkdir -p public
          rsync -av --exclude=public --exclude='.*' . ./public/
          
      - name: deploy-oss
        env:
          OSS_AKID: ${{ secrets.OSSAKID }}
          OSS_AKSECRET: ${{ secrets.OSSAKSECRET }}
          OSS_BUCKET: ${{ secrets.OSSBUCKET }}
          LOCAL_DIR: ./public/
          REMOTE_DIR: /
        run: |
          wget https://gosspublic.alicdn.com/ossutil/1.7.19/ossutil-v1.7.19-linux-amd64.zip || { echo "Download failed"; exit 1; }
          unzip ossutil-v1.7.19-linux-amd64.zip || { echo "Extracting failed"; exit 1; }
          chmod +x ./ossutil-v1.7.19-linux-amd64/ossutil64 || { echo "Permission setting failed"; exit 1; }
          echo "${OSS_AKID} is operating"
          ./ossutil-v1.7.19-linux-amd64/ossutil64 -e oss-cn-beijing.aliyuncs.com -i ${OSS_AKID} -k ${OSS_AKSECRET} cp -rf ${LOCAL_DIR} oss://${OSS_BUCKET}${REMOTE_DIR}
          echo 'Finished.'
```


