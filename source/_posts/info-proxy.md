---
title: 使用 Node.js 建立 info 的简单代理服务器
date: 2020/2/15 12:06
update: 2020/06/26 13:40
url: info-proxy.html
tags:
- NodeJs
- Proxy
categories:
- Coding
---

主要需求是在校外结合 RSSHub 订阅 info ，避免错过重要通知。

<!--more-->

首先利用清华大学提供的 [WEBVPN](https://webvpn.tsinghua.edu.cn/) 搭建实现自动登录功能的代理服务器。（新版 WEBVPN 只是校内代理，相较于以前自动登录 info 的 SSLVPN 风险较小）

学习并使用 Node.js，使用单语句<code>req.pipe(request).pip(res)</code>实现不完美的网页代理（在这里够用了）。

```javascript
var request = require("request");
var express = require("express");

//登陆 post 地址
let url = 'https://webvpn.tsinghua.edu.cn/do-login?local_login=true';
//登陆的用户邮箱和密码
let user = {
	username: '',
	password: '',
};
//登陆 post 的所有数据
let datas = {
	auth_type: 'local',
	username: user.username,
	sms_code: '',
	password: user.password,
	remember_cookie: 'on',
};

//设置头部
let ua = `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36`;
let headers = {
	'User-Agent': ua,
};

let opts = {
	url: url,
	method: 'POST',
	headers: headers,
	form: datas,
};

//模拟登陆
var cookie;
request(opts, (e, r, b) => {
	cookie = r.headers['set-cookie'];
});

var app = express();
app.use("/", function (req, res) {
	request({
			url: "https://webvpn.tsinghua.edu.cn/http/77726476706e69737468656265737421f9f9479369247b59700f81b9991b2631506205de/initial/letter.gif",
			headers: {
				Cookie: cookie, 
				'User-Agent': ua,
			}},(e, r, b) => {
				//检测 cookie 是否失效
				if(b.indexOf('html')>0){
					request(opts, (e, r, b) => {
						cookie = r.headers['set-cookie'];
					});
				}
			}
	);
	//增强安全性
	if(req.url.match(/^\/(http|wengine-vpn)/)){
		req.pipe(request({
				url: "https://webvpn.tsinghua.edu.cn" + req.url,
				headers: {
					Cookie: cookie, 
					'User-Agent': ua,
				},
				encoding: null,
				qs: req.query,
				method: req.method
			})).pipe(res);
	}
	});
app.listen(process.env.PORT || 3000);
```

同时使用 [RSSHub](https://docs.rsshub.app/university.html#qing-hua-da-xue) 实现了清华大学校内信息发布平台的 RSS 化。
