---
title: 博客自动化部署(Git + GitHub Action + VPS)
date: 2024-02-28 22:10:02 +0800
categories: [开始, 自动化部署]
tags: [Git, GitHub Actions, VPS]     # TAG names should always be lowercase
mermaid: true
math: true
---

> 项目自动化

### 实现逻辑

- 本地Git提交 $\rightarrow$ $$\frac{编译代码 \rightarrow VPS}{GitHub Action}$$ $\rightarrow$ VPS执行脚本并重启服务

