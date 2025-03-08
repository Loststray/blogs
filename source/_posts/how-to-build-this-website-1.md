---
title: 网站构建记录
date: 2025-03-06 15:02:50
tags: nodejs git GithubAction
categories: 教程
comments: true
---

你现在正在访问的博客网站是基于hexo的，GithubActions作为脚本驱动的，托管在Github page上的一个静态网页。

这是一篇关于这个网站是如何构建的教程，希望这个教程能帮助你建立自己的博客网站。

# 1 前置要求

首先，你需要有一个github账号，如果要本地部署还需要下载 [git](https://git-scm.com/downloads) , [Node.js](https://nodejs.org/zh-cn) 和他们的依赖项。

安装完成后打开终端进行测试

```bash
git -v
node -v
```

如果能成功返回版本则说明安装成功。

## (可选) npm换源

如果npm下载速度慢，可以考虑更换npm的下载源来加速

```bash
默认源：
npm config set registry https://registry.npmjs.org/
淘宝源：
npm config set registry https://registry.npmmirror.com
阿里云：
npm config set registry https://npm.aliyun.com
腾讯云：
npm config set registry http://mirrors.cloud.tencent.com/npm/
```

# 2 hexo 安装

从[hexo](https://hexo.io/)官网进入，终端输入

```bash
npm install -g hexo-cli
hexo -v
```

如果显示出hexo的版本说明安装成功。

# 3 github pages 创建

如果你已经有一个公网可用的域名和服务器可以跳过这一步

在github上创建一个仓库`你的用户名.github.io`，你的网站就会部署到`你的用户名.github.io` (特别注意，如果你的github用户名有大写字母，创建仓库的时候需要改成小写)

或者直接创建一个仓库 `你的仓库名` 然后在 `Settings-Pages-Source` 里面选择 Github Actions，你的网站应该部署到`你的用户名.github.io/你的仓库名`。

# 4 git 链接 github

接下来配置git的连接，打开git bash运行以下代码

```bash
git config --global user.name `你的github用户名`
git config --global user.email `你的github邮箱`
```

运行这两行代码之后git所有的提交都会附带你的用户名和邮箱。

如果不想要每次git push都输入github账号密码，可以在本机配置ssh rsa key，将账号和主机关联起来具体操作如下

```bash
ssh-keygen -t rsa -C `你的github邮箱`
```

```bash
Generating public/private rsa key pair.
# Enter file in which to save the key (/c/Users/`你的用户名`/.ssh/id_rsa): [Press enter]
```

此处操作是生成一对密钥，其中之一(id_rsa)是私钥，另一个(id_rsa.pub)是公钥

这里可以自定义存储的文件名，也可以直接用默认名 (如果不使用默认文件名则之后必须手动将 _私钥_ 加入到ssh-agent，如果ssh-agent没有启用则需要手动启用)

```bash
Windows:
# start the ssh-agent in the background
Get-Service -Name ssh-agent | Set-Service -StartupType Manual
Start-Service ssh-agent
ssh-add c:/Users/`你的用户名`/.ssh/你的私钥
```

登录你的github账号，进入[SSH and HPH keys](https://github.com/settings/keys) 点击右上角的 `New SSH key`，把 _公钥_ 文件的内容复制进去

用以下代码进行测试，如果出现连接到账号说明配置成功

```bash
ssh -T git@github.com
```

# 5 新建 hexo 项目

接下来把你刚刚创建的仓库 `git clone` 下来。

```bash
git clone https://github.com/`你的用户名`/`你的仓库`.git
```

接下来创建hexo项目

```bash
hexo init `clone的项目`
cd `clone的项目`
npm install
```

# 6 配置 _config.yml

这里可以参考[hexo官方文档](https://hexo.io/zh-cn/docs/configuration)

配置完成后可以在本地部署

```bash
hexo clean | hexo generate | hexo serve
```

默认部署在 [http://localhost:4000/](http://localhost:4000/)

# 7 配置主题和插件

一个好的网站少不了好看的主题和各种插件，这里推荐前往[hexo官方网站](https://hexo.io/)查找插件和主题并且根据配置文档自行操作

# 8 部署到 Github Pages

可以具体参考[官方文档](https://hexo.io/zh-cn/docs/github-pages)

主要步骤就是配置`.gitignore`，和编写工作流

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
*.code-workspace
```

这里使用的`node.js`版本应该和本地主机匹配

```yml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

接下来访问目标网站 `username.github.io` 查看

# 总结

完成以上步骤后，您的 hexo 博客就搭建完成了！主要实现了:

- 基础环境配置
- Hexo 框架安装
- GitHub Pages 部署
- 主题美化