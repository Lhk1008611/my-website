---
slug: github-page-blog-post
title: GithubPage Blog Post
authors:
  name: Lhk1008611
  title: Docusaurus Core Team
  url: https://github.com/Lhk1008611
  image_url: https://avatars.githubusercontent.com/u/76029401?v=4
tags: [github-page, Lhk1008611]
---
# github_page的配置
1. 创建一个 username.github.io 的仓库
2. 将本地静态页面文件推送到这个仓库
3. 打开username.github.io这个网站即可访问

# CMS
- 内容管理系统

# Docusaurus
- 一个开源的静态内容管理系统，可以快速构建一个静态网站
- 官网：https://docusaurus.io/
- 我们可以通过使用 Docusaurus 这个框架打造出一个具有个人风格的网站，并配合github-page这样一个功能，将静态页面部署在github上
- 使用
    1. 安装
        ```bash
         npx create-docusaurus@latest my-website classic
        ```
    2. 启动
        ```bash
        npm start | yarn start
        ```
    3. 在`docusaurus.config.js`里面配置页面信息
    4. 构建
        ```bash
        npm run build | yarn build
        ```
    5. 部署在github上
        - 首先在`docusaurus.config.js`文件中配置部署在哪一条分支
            ```js
                deploymentBranch:'main'
            ```
        - 配置环境变量 `GIT_USER`,值为github的用户名
        - 执行部署命令
            ```bash
                npm run deploy | yarn deploy
            ```
    - 在pages目录下可以写自己的页面（推荐写js文件，基于react）
    - 在blog目录下可以写md文件，用于作为blog
    - 在docs目录下可以创建文件夹作为文档