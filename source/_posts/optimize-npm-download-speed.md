---
title: 给你的 npm 提提速——优化国内 npm 下载速度
author: 子丶言
date: 2020-07-10 11:10:18
tags: ['npm', 'yarn', 'Node.js']
categories: ['npm']
---

在国内大环境的背景下，一些外国网站的访问速度经常会遇到加载慢的情况。这对于前端开发人员而言，最难接受的就是 npm 包下载过慢的问题。你想想呐，一个程序员很开心的写着代码，喝着快乐水，却在下载 npm 依赖时频繁遭遇 `Network Error` 或 `Network Timeout`。你那时还有心情写代码么？为了解决广大同僚的工作问题，我有一法可解。
<!-- more -->

## 更换 npm registry 源

Node.js 的 npm cli 默认的 registry 源是 `https://npmjs.com/`。由于某些原因，[npmjs.com](https://npmjs.com/) 在国内的访问速度很不稳定。所以我们可以考虑用国内的镜像网站地址替换默认的地址。

### 查看当前 registry 源

```shell
yarn config get registry
# 或
npm config get registry
```

这条命令会输出你当前系统的 npm registry 源的网址。如果你显示的是 `https://registry.npmjs.com/` 那么你可以考虑更换为更快的国内 registry 源。

目前国内比较稳定的镜像源：
```
taobao --- https://registry.npm.taobao.org/
cnpm --- https://r.cnpmjs.org/
nj --- https://registry.nodejitsu.com/
rednpm --- https://registry.mirror.cqupt.edu.cn/
npmMirror --- https://skimdb.npmjs.com/registry/
deunpm --- http://registry.enpmjs.org/
```

### 如何设置

个人推荐使用淘宝的镜像源 `https://registry.npm.taobao.org/`

#### 临时设置

```shell
yarn add node_module --registry https://registry.npm.taobao.org/
# 或
npm install node_module --registry https://registry.npm.taobao.org/
```

#### 全局设置

```shell
yarn config set registry https://registry.npm.taobao.org/
# 或
npm config set registry https://registry.npm.taobao.org/
```

#### 使用第三方脚本快速修改、切换 npm 镜像源

你可以考虑使用 [nrm](https://www.npmjs.com/package/nrm)。

```shell
yarn global add nrm
# 或
npm install -g nrm
```

```shell
# 查看当前可用的所有镜像源
nrm ls
# 切换淘宝源
nrm use taobao
# 测试速度
nrm test taobao
```

### 更快的 npm 资源

你以为切换成了国内源，npm 就没有优化空间了么？你想变得更快更强么？那么请你继续往下看。

```shell
npm set disturl https://npm.taobao.org/dist # node-gyp 编译依赖的 node 源码镜像

## 以下选择添加
npm set sass_binary_site https://npm.taobao.org/mirrors/node-sass # node-sass 二进制包镜像
npm set electron_mirror https://npm.taobao.org/mirrors/electron/ # electron 二进制包镜像
npm set puppeteer_download_host https://npm.taobao.org/mirrors # puppeteer 二进制包镜像
npm set chromedriver_cdnurl https://npm.taobao.org/mirrors/chromedriver # chromedriver 二进制包镜像
npm set operadriver_cdnurl https://npm.taobao.org/mirrors/operadriver # operadriver 二进制包镜像
npm set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs # phantomjs 二进制包镜像
npm set selenium_cdnurl https://npm.taobao.org/mirrors/selenium # selenium 二进制包镜像
npm set node_inspector_cdnurl https://npm.taobao.org/mirrors/node-inspector # node-inspector 二进制包镜像
npm set selenium_cdnurl=http://npm.taobao.org/mirrors/selenium
npm set node_inspector_cdnurl=https://npm.taobao.org/mirrors/node-inspector
```

npm 和 yarn 都会对 npm 进行本地缓存，如果你的资源依然被解析为了 `https://registry.npmjs.com/`，那么可以考虑使用以下命令进行缓存清理：

```shell
yarn cache clean
# 或
npm cache clean --force
```
