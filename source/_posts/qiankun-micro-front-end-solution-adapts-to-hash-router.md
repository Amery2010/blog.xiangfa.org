---
title: qiankun 微前端方案适配 HashRouter 模式及 TypeScript 项目
author: 子丶言
date: 2021-01-25 17:43:08
tags: ['qiankun']
categories: ['qiankun']
---

微前端的概念在近几年不断“升温”，越来越多的公司尝试通过微前端方案来解决“巨石”应用([Frontend Monolith](https://www.youtube.com/watch?v=pU1gXA0rfwc))的问题。微前端的六种常见实现方案不是本文的讨论点，感兴趣的朋友可以阅读[微前端在小米 CRM 系统的实践](https://xiaomi-info.github.io/2020/04/14/fe-microfrontends-practice/)这篇文章。在比较文中的各类实现方案后，我确定了使用 [Qiankun](https://qiankun.umijs.org/zh/guide) 来实现前端微服务化，但在实现过程中遇到了一些困难，本文主要描述问题及解决方案。
<!-- more -->

### 问题一：支持 HashRouter 模式

Qiankun 官网的适配方案使用了 BrowserRouter 模式，但对于如何适配 HashRouter 模式并没有展开描述，不管我如何调整，路由似乎并没有被正确的激活，这曾让我数次决定放弃 qiankun 微前端方案，但 Single-Spa 的适配调整又把我再次劝退...
我一开始一直以为是在 HashRouter 的配置上出了问题，尝试将 basename 设为 `window.__POWERED_BY_QIANKUN__ ? '#/demo' : '#/'`，依然无法解决项目加载的问题。从项目的错误提示中，我摸到了些许蛛丝马迹，将问题从子应用回归到主应用的配置上，原来 `activeRule` 是支持 Hash 前缀的，比如 `activeRule: '#/demo'`，而这一点官网并没有任何提示！至此，HashRouter 的适配问题迎刃而解，正所谓“山穷水复疑无路，柳暗花明又一村”。

```ts
import { registerMicroApps, start } from 'qiankun'

registerMicroApps([
  {
    name: 'reactApp',
    entry: '//localhost:3000',
    container: '#container',
    activeRule: '#/demo',
  },
])
// 启动 qiankun
start()
```

### 问题二：适配 TypeScript 项目

官方的文档是基于 JavaScript 写的，如果直接将 JavaScript 代码复制到 TypeScript 项目时，你将会收获一堆的错误提示，比如以下适配代码简直是 TypeScript 项目的“灾难”！

```js
if (window.__POWERED_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

以上代码的问题在于，一个全局变量未定义，两个 `window` 上的变量未定义，对于这类 “Dirty Code”，你可以通过补充定义进行修复，具体代码如下：

```ts
interface Window {
  __POWERED_BY_QIANKUN__?: string
  __INJECTED_PUBLIC_PATH_BY_QIANKUN__?: string
}

declare let __webpack_public_path__: string | undefined
```

**注意**：以上代码最好放到全局引入的 TypeScript 定义文件中，比如 `types.d.ts` 或 `react-app-env.d.ts` 这类文件中。


### 问题三：如何适配 customize-cra 插件

Qiankun 官网对 React 项目的适配方案中对 webpack 配置调整提到了使用 `@rescripts/cli` 这个插件，但对于老项目而言调整 webpack 插件显然存在很大的项目风险。我司的项目使用了 `react-app-rewired + customize-cra` 的方案，但网上似乎并没有针对这个问题的具体解决方案。因为我之前写过些许自定义的 customize-cra plugin，因此这个问题相对容易解决，具体实现代码如下：

```js
const pkg = require('./package.json')
const { override, overrideDevServer } = require('customize-cra')

// 适配 qiankun 微前端逻辑
const adaptMicroApp = () => {
  return (config) => {
    /**
     * 根据 qiankun 文档配置，官方文档里使用的是 `${pkg.name}-[name]` 的形式，
     * 但我个人觉得这个形式更像是一种笔误...直接使用 `pkg.name` 并不会影响实际效果。
     */
    config.output.library = pkg.name
    config.output.libraryTarget = 'umd'
    config.output.jsonpFunction = `webpackJsonp_${pkg.name}`
    config.output.globalObject = 'window'
    return config
  }
}

// 开启项目跨域模式
const adaptCors = () => {
  return (config) => {
    // 关闭主机检查，使微应用可以被 fetch
    config.disableHostCheck = true
    // 配置跨域请求头，解决开发环境的跨域问题
    config.headers = {
      'Access-Control-Allow-Origin': '*',
    }
    // 如果你的路由方案是 BrowserRouter，需要配置 history 模式
    // config.historyApiFallback = true
    return config
  }
}

module.exports = {
  webpack: override(
    // 适配 qiankun 微前端逻辑
    adaptMicroApp()
  ),
  devServer: overrideDevServer(
    // 开启项目跨域模式
    adaptCors()
  ),
}
```

以上几个就是我目前遇到的比较难解的几个问题及我的解决方案，希望能给你带来些许帮助！
