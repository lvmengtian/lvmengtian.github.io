---
title: vuepress部署Github Pages
createTime: 2025-08-22T10:40:17.000Z
permalink: /notes/2w5wcxxz/
---

## 部署
### 创建workflow文件
路径：git仓库 -> Actions -> New workflow -> Simple workflow

内容配置如下，并且文件命名后缀为`yml`
```
name: docs

on:
  # 每当 push 到 main 分支时触发部署
  push:
    branches: [main]
  # 手动触发部署
  workflow_dispatch:

jobs:
  docs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          # “最近更新时间” 等 git 日志相关信息，需要拉取全部提交记录
          fetch-depth: 0

      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          # 选择要使用的 pnpm 版本
          version: 10
          # 使用 pnpm 安装依赖
          run_install: true

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          # 选择要使用的 node 版本
          node-version: 22
          # 缓存 pnpm 依赖
          cache: pnpm

      # 运行构建脚本
      - name: Build VuePress site
        run: pnpm docs:build

      # 查看 workflow 的文档来获取更多信息
      # @see https://github.com/crazy-max/ghaction-github-pages
      - name: Deploy to GitHub Pages
        uses: crazy-max/ghaction-github-pages@v4
        with:
          # 部署到 gh-pages 分支
          target_branch: gh-pages
          # 部署目录为 VuePress 的默认输出目录
          build_dir: docs/.vuepress/dist
        env:
          # @see https://docs.github.com/cn/actions/reference/authentication-in-a-workflow#about-the-github_token-secret
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
### 配置Pages
路径：git仓库 -> Settings -> Code and automation -> Pages -> Build and deployment
- source：选择Deploy from a branch
- branch：选择gh-pages    文件 /root 

## 遇到的问题
### 问题1
部署失败，提示报错信息如下：
```
Pushing docs/.vuepress/dist directory to gh-pages branch on xxx/xxx.github.io repo
  /usr/bin/git push --force ***github.com/xxx/xxx.github.io.git gh-pages
  remote: Permission to xxx/xxx.github.io.git denied to github-actions[bot].
  fatal: unable to access 'https://github.com/xxx/xxx.github.io.git/': The requested URL returned error: 403
  Error: The process '/usr/bin/git' failed with exit code 128
```
**原因：** workflow的权限不够

**解决方式：**
路径：git仓库 -> Settings -> Code and automation -> Actions -> General -> Workflow permissions
打开：`Read and write permissions`


## 参考链接
1. [vuepress deployment](https://vuepress.vuejs.org/guide/deployment.html#github-pages)

