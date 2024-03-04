---
title: 博客自动化部署(Git + GitHub Action + VPS)
date: 2024-02-28 22:10:02 +0800
categories: [开始, 自动化部署]
tags: [Git, GitHub Actions, VPS, SSH]     # TAG names should always be lowercase
mermaid: true
math: true
render_with_liquid: false
---

> 项目自动化

### 实现逻辑

- 本地Git提交 $\rightarrow$ $$\frac{编译代码 \rightarrow VPS}{GitHub Action}$$ $\rightarrow$ VPS执行脚本并重启服务

### 前期准备

> 准备ssh免密登录的private key，我这里直接用我自己机器的key，建议重新生成一个
{: .prompt-info }

``` bash
ssh-keygen -t rsa -C "github actions"
```

- 在github的仓库中设置思考，处于安全性的考虑，使用GitHub的Secrets。 在项目的Settings->Secrets->Actions增加私钥，我这边将服务器的Host，Username都配置进去了，方便修改。

### Actions文件

> 存储在项目目录的.github/workflows/deploy.yml，文件名可以随便起，路径必须对。
{: .prompt-warning }

actions文件示例如下：

``` yml
name: CICD
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
  workflow_dispatch:
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        ruby-version: ['3.2.2']
    steps:
    - uses: actions/checkout@v3
    - name: Set up Ruby
      uses: ruby/setup-ruby@55283cc23133118229fd3f97f9336ee23a179fcf # v1.146.0
      with:
        ruby-version:  ${{ matrix.ruby-version }}
        bundler-cache: true # runs 'bundle install' and caches installed gems automatically

    - name: Build
      run: |
        bundle exec jekyll build

    - name: copy file via ssh password
      uses: cross-the-world/scp-pipeline@master
      with:
        host: ${{ secrets.DEPLOY_HOST }}
        user: ${{ secrets.DEPLOY_USER }}
        pass: ${{ secrets.DEPLOY_PASSWORD }}
        key: ${{ secrets.DEPLOY_SECRET }}
        local: "./_site"
        remote: "/var/www"

    - name: executing remote ssh commands to develop
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.DEPLOY_HOST }}
        username: ${{ secrets.DEPLOY_USER }}
        password: ${{ secrets.DEPLOY_PASSWORD }}
        key: ${{ secrets.DEPLOY_SECRET }}
        script: sh /home/deploy.sh
```
