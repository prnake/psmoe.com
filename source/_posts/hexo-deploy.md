---
title: 使用 Github Actions 将 Hexo 托管至腾讯云
categories:
	- Tools
date: 2020/7/8 15:45
---

上车了腾讯云9.9元包年（500G存储+2000G流量）的[静态网站托管](https://cloud.tencent.com/product/wh)活动，于是考虑使用 `GitHub Actions` 实现 `Hexo` 的自动化生成和部署，以及利用腾讯云和 `Github Pages` 实现国内外CDN加速。

<!--more-->

首先创建新项目，将 `Hexo` 目录下的必要文件上传至 `master` 分支（`Hexo` 应该会自动生成 `.gitignore` 文件）。

然后利用`GitHub Actions` 自动化生成静态文件并部署至 `gh-pages` 分支。

在 `blog` 仓库的 `Actions` 选项卡下点击新建 `workflow` ，编写如下配置。

```yaml
name: Deploy Blog

on: [push] # 当有新push时运行

jobs:
	build:
		runs-on: ubuntu-latest # 在最新版的Ubuntu系统下运行
		
		steps:
		- name: Checkout # 将仓库内master分支的内容下载到工作目录
			uses: actions/checkout@v1
			
		- name: Setup Node.js # 配置Node环境
			uses: actions/setup-node@v1
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

每当有新的 push 发生时，都会触发 `GitHub Actions` ，生成静态文件并 push 至 `gh-pages` 分支。

同时在 Hexo 的 `source` 目录下添加 `CNAME` 文件，填入需要解析到的域名，这样每次 `gh-pages` 变化时 github 都会自动运行 `Page Build` 并设置解析。

接下来使用 [Tencent CloudBase Github Action](https://github.com/TencentCloudBase/cloudbase-action) 将 `gh-pages` 分支部署到腾讯云开发平台，这里的思路是检测 `Page Build` 的动作。

创建一个新的 `workflow` ，编写如下配置。

```yaml
name: Tencent CloudBase

on: [page_build]

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Tencent Cloudbase
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: gh-pages
      - name: Deploy static to Tencent CloudBase
        id: deployStatic
        uses: TencentCloudBase/cloudbase-action@v1.1.1
        with:
          secretId: ${{ secrets.SECRET_ID }}
          secretKey: ${{ secrets.SECRET_KEY }}
          envId: ${{ secrets.ENV_ID }}
          staticSrcPath: ./
```

其中的 `SECRET_ID` 和 `SECRET_KEY` 在腾讯云[访问管理](https://console.cloud.tencent.com/cam/capi)页面获取，`ENV_ID` 即环境Id，在云开发的[控制台](https://console.cloud.tencent.com/tcb/env/overview)获取，并添加到 `setting/secret` 中。

这样，每当有新 push 时， github 都会自动生成 Hexo 的静态博客文件并部署到腾讯云和 Github Pages 。

由于腾讯云CDN缺乏国外服务器，所以国外线路走 Github 自己的 CDN 更好。使用 DNS 服务（例如[DNSPod](https://www.dnspod.cn/)）将域名国内 CNAME 至腾讯云，国外 CNAME 至 Github 平台即可。

![CNAME](/images/hexo-deploy1.png)

最后进行一下延迟测试。

![PING TEST](/images/hexo-deploy2.png)

总的返回ip有18个，其中 github 4个，腾讯云14个，全球解析延迟基本控制在30ms。相比之前使用的七牛云延迟应该有所上升，但网站加载时的首字节时间（TTFB）大幅下降，因此网站的实际访问体验明显变快了（虽然为了加载字体和渲染Live2D依然要耗费大量时间），这估计与回源效率和服务器质量有关。

总的部署可以参考[本博客的 Github 项目](https://github.com/prnake/psmoe.com)。

