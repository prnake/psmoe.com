---
title: 使用 GitHub Actions 自动化生成 Hexo 博客
categories:
	- Tools
date: 2020/7/2 21:17
---

使用 GitHub Actions ，最大程度降低本地环境要求和开销。

show_me_more

在 `blog` 仓库的 `Actions` 选项卡下点击新建 `workflow` ，编写如下配置。

```yaml
name: Deploy Blog

on: [push] # 当有新push时运行

jobs:
	build:

		runs-on: ubuntu-latest # 在最新版的Ubuntu系统下运行
		
		steps:
		- name: Checkout # 将仓库内master分支的内容下载到工作目录
			uses: actions/checkout@v1 # 脚本来自 https://github.com/actions/checkout
			
		- name: Setup Node.js # 配置Node环境
			uses: actions/setup-node@v1 # 配置脚本来自 https://github.com/actions/setup-node
			with:
				node-version: 12
		
		- name: Setup Hexo
			env:
				ACTION_DEPLOY_KEY: ${{ secrets.ACTION_DEPLOY_KEY }} 
				#在 setting/secret 下添加名称为 ACTION_DEPLOY_KEY 的私钥
			run: |
				# set up private key for deploy
				mkdir -p ~/.ssh/
				echo "$ACTION_DEPLOY_KEY" | tr -d '\r' > ~/.ssh/id_rsa # 配置秘钥
				chmod 600 ~/.ssh/id_rsa
				ssh-keyscan github.com >> ~/.ssh/known_hosts
				# set git infomation
				git config --global user.name 'prnake'
				git config --global user.email 'prnake@gmail.com'
				# install dependencies
				npm i -g hexo-cli # 安装hexo
				npm i
	
		- name: Deploy
			run: |
				# publish
				hexo generate && hexo deploy # 执行部署程序
```

同时在 `hexo` 的配置文件 `_config.yml` 中添加 `deploy` 相关的参数，例如：

```yaml
deploy:
	type: git
	repo: git@github.com:prnake/psmoe.com.git
	branch: gh-pages
```

之后再绑定相应的 `Github Pages` 即可。

这样，每当有新 `push` 时， `github` 都会自动生成 `hexo` 的静态博客文件并发布。

