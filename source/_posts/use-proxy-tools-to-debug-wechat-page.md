---
title: 使用代理映射解决微信页面调试难题
author: 子丶言
date: 2020-06-09 15:03:57
tags: ['微信页面开发', '微信页面调试', '微信开发者工具']
categories:
  - 微信页面开发
---

在开发微信页面过程中，有些人可能会遇到跟我相同的麻烦——如何快速调试页面。在微信页面中获取数据（比如，用户昵称和头像）都需要走微信的授权流程。在微信授权之后你可以拿到一个 code，用这个 code 你可以经由后端获取到用户数据。但微信 code 的使用是存在限制的，比如每个 code 只能使用一次，微信 code 获取的域名有白名单限制等。这篇文章介绍了一种在不调整后端逻辑的情况下，解决获取微信 code 过程中的域名白名单验证的问题。
<!-- more -->

## 解决微信 code 域名白名单验证的问题

目前微信页面授权采用的是 OAuth2 的方案。微信会对授权的域名做白名单校验，这是一种安全的做法，但也是一种让开发人员~~抓狂~~头疼的做法。但这只是一种简单的安全校验规则，你可以通过域名代理映射来解决这个问题。

### 通过 Hosts + Nginx 实现代理映射

如果你项目的端口号是 80，你可以通过简单的修改 hosts 进行本地域名的映射。但 hosts 只能解决 `http://xxxx.com` 这种形式的域名映射，对于非 80 端口的本地服务，则无能为力。

我们在本地调试过程中，可能会用到 IP 或本地域名，而项目开发框架为了避免项目间的冲突，因此很少会直接使用 80 端口。因此你经常可以看到类似于 8080 这样的端口号。

![项目启动](https://gaeacdn.jiliguala.com/devjlgl/tmp/7640f6dd39389c0e500248fe1aaf4220.png)

这类本地服务你无法直接通过修改 hosts 解决微信 code 域名白名单验证的问题。这时候你就需要通过代理来解决问题。

最简单的代理方案就是通过 Nginx 来做代理转发。

在设置 Nginx 配置之前，你需要先通过增加 hosts 映射来“捕获”微信开发者工具的域名请求。

```
# 假定你的 Nginx 服务使用 127.0.0.1，且端口号为 80
127.0.0.1 	www.yourdomain.com
```

之后你可以在 Nginx 配置中添加以下代码，用以注册服务。

```nginx
server {
  listen          80;
  server_name     www.yourdomain.com; #修改为你需要调试的域名

  location / {
    index  index.html index.htm index.php;
    index  proxy_set_header Host $host;
    index  proxy_set_header X-Real-IP $remote_addr;
    index  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://localhost:8080; #你本地的项目域名
  }
}
```

然后重启 Nginx 服务，这样你就可以在微信开发者工具里调试本地代码了。

虽然解决了目前遇到的问题，但这种方案操作比较复杂，重置过程相对繁琐，可以用来解决一时需求，无法作为长期解决方案。除此之外，在 Mac 操作系统上，由于系统安全限制，Nginx 无法直接使用 80 端口服务。

### 通过代理软件实现代理映射

市面上的代理软件众多，此处不展开讨论，此文以 Mac 平台的 Charles 为例，为大家提供一种使用思路。

#### 配置 Charles 代理

首先你需要勾选 Charles 的自动代理 `Proxy -> macOS Proxy`，并设置远程映射 `Tools -> Map Remote`

![远程映射配置](https://gaeacdn.jiliguala.com/devjlgl/tmp/f285555e5e02ec872ffd4f838594340f.png)

你点击 `Add` 可以增加新的规则，如果 `Add` 按钮灰色不可点击，你需要勾选 `Enable Map Remote`，之后你就可以编写你的映射规则了。

![配置映射规则](https://gaeacdn.jiliguala.com/devjlgl/tmp/1e34f34bc6d6031d48cd6361e24a73e7.png)

#### 配置微信开发者工具代理

Charles 默认不会代理系统的网络请求，只是启动了一项全局代理服务，所以你还需要通过配置微信开发者工具的代理设置才可以真正的使用 Charles 代理。

在微信开发者工具中找到 `设置 -> 代理设置 -> 代理`，然后选择"手动代理"，我这里设置为 `locolhost 8888`，Charles 默认的代理端口号为 `8888`，你可以在 Charles 修改默认代理端口号。

这样你就可以在微信开发者工具里用本地服务模式进行开发了。

最新的 Chales 版本默认执行白名单机制，如果你未在白名单设置内进行访问域名配置，可能会遇到微信开发者工具页面打开异常的情况，你可以通过 `Tools -> Allow List` 进行关闭。
