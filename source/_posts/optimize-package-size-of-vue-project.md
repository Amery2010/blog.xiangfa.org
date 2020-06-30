---
title: 优化 vue 项目包大小，提升首屏加载速度
author: 子丶言
date: 2020-06-29 18:13:40
tags: ['vue', 'vue-cli', 'webpack', '项目优化']
categories:
  - Vue
---

在前端性能优化中有一个很重要的概念-白屏时间。所谓白屏时间，即用户点击一个链接或打开浏览器输入URL地址后，从屏幕空白到显示第一个画面的时间。**白屏时间的长短将直接影响用户对网站的第一印象。**白屏时间过长还会导致用户流失，降低页面的留存率。

公司的 To C 项目目前采用 vue 开发，使用 vue-cli 构建的项目已经做了不少性能优化，但随着项目的迭代，依然逃脱不了项目白屏时间过长的问题。为了缩短首页白屏时间，我开始了首屏分包加载试验，以下是我的首屏加载优化心得，希望对你有所帮助。
<!-- more -->

## 页面按需加载

目前公司内部不少 vue 项目都使用 vue-router 这个库，vue-router 可以借助 webpack 的 `import()` 动态导入实现页面分包，从而实现页面的按需加载。

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from './views/Home'

Vue.use(VueRouter)

const router = new VueRouter({
  routes: [
    {
      path: '/home',
      name: 'home',
      component: Home
    },
    // 使用 import() 可以实现页面的动态载入，也就是不会和其他页面内容打包在一起。
    // 其中 webpackChunkName 是 webpack 用于命名分包的名称的。
    {
      path: '/',
      name: 'landing',
      component: () => import(/* webpackChunkName: "landing" */ './views/Landing')
    }
  ]
})

export default router
```

如果你的项目页面较多时，你会看到 vue-cli 构建后会多出很多已命名的分包，而首屏页面的分包体积也会明显的变小。这样首屏页面初始化只需要下载比之前小一些的 js 文件就可以马上显示了，从而减少首屏的白屏时间。

此时，你可能会有另一个担忧：如果其他页面都是按需加载的情况，那么页面切换会不会变慢？这是一个好问题！那么到底会不会出现这个情况呢？答案是可能会，但你不需要担心，因为 webpack 在打包时已经考虑到了这个问题，分包并非在页面载入时才去加载，而是通过 preload 或则 prefetch 的方式进行了预加载。所以在页面初始化之后的一段时间，页面的其他分包 js 会被陆续加载进来，在页面切换时不会出现需要等待加载的情况。

## 分离项目中的基础依赖

你在开发 vue 项目过程中，可能会使用到 Vue、Vuex、VueRouter 以及 axios 等常用依赖。这些依赖在项目开发过程中以基础依赖的形式存在。而每次项目打包都会把这些基础依赖重复打包进项目，以至于即便是最简单的页面也会生成不小的 js 包文件。如果我们将这些基础依赖抽离出来，这样每次项目更新，用户只需更新业务相关的 js 文件就可以正常显示页面了。

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.11/dist/vue.runtime.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vuex@3.5.1/dist/vuex.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue-router@3.3.4/dist/vue-router.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios@0.19.2/dist/axios.min.js"></script>
```

```js
// vue.config.js
module.exports = {
  configureWebpack: config => {
    config.externals = {
      'vue': 'Vue',
      'vuex': 'Vuex',
      'vue-router': 'VueRouter',
      'axios': 'axios'
    }
  }
}
```

当你将项目中的基础依赖进行分离之后你会发现 vue-cli 生成的项目体积明显变小了，首屏加载速度在第一次加载时并没有明显变化，但在之后的加载过程中会得到显著的提升。

### 处理小众依赖问题

对于私有依赖或者小众的依赖，你如果也想将其分包处理，但你却无法从网上找到对应的 cdn 链接的时候，你可以考虑自己动手打包。

只有 `amd` 或者 `umd` 格式的 js 文件才可以通过 `script` 标签的方式进行引入，很多 npm 依赖官方并不会构建 umd 格式的依赖包，这种情况下，你需要自己想办法生成。你可以考虑通过 rollup 来生成 umd 格式的依赖包，具体做法如下：

#### 引入 rollup 相关依赖

```shell
yarn add rollup rollup-plugin-uglify @rollup/plugin-node-resolve @rollup/plugin-commonjs -D
// 或者使用 npm
npm install rollup rollup-plugin-uglify @rollup/plugin-node-resolve @rollup/plugin-commonjs --save-dev
```

#### 引入依赖并导出文件

```js
// src/demo.js
import { default as demo } from 'demo'

export default demo
```

#### 创建 rollup.config.js 文件

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'
import { uglify } from 'rollup-plugin-uglify'

export default [
  {
    // 源文件路径
    input: 'src/demo.js',
    context: 'window',
    output: {
      // 生成的目标文件路径及文件名称
      file: 'libs/demo.min.js',
      // umd 导出的全局对象名称
      name: 'demo',
      format: 'umd',
      // 是否生成 sourceMap
      sourcemap: true,
    },
    plugins: [
      // 用于解析 node_modules 依赖关系
      resolve(),
      // 用于处理 commonjs 导入导出格式
      commonjs(),
      // 压缩 js 文件
      uglify(),
    ],
  }
]
```

最后你只需要在 package.json 中添加 script `build-lib: "rollup -c"`，运行 `npm run build-lib` 就可以生成 umd 格式的依赖包了。

自己构建的 umd 格式的依赖包可以放进项目中进行引入，但比较推荐的方案是上传到 cdn 服务上，然后引入 cdn 的链接。

### 独立依赖按需载入

在有一些子页面中，你可以会引入一些独立的依赖文件，所谓独立依赖，是指那些只在特定页面或组件中使用的依赖，比如 qrcode。qrcode 依赖默认会被打包进项目依赖中，因此会增大项目依赖包的体积，对于普通用户来说，很可能并不会用到。如果可以想办法将这个依赖分离出去，然后按需载入的话，可以进一步减少基础依赖包的体积。

独立依赖的按需载入依然是借助了 webpack 的动态导入来实现。

常规的依赖使用方式：

```js
import QRcode from 'qrcode'

QRcode.toDataURL('xxxx', {
  margin: 2,
  width: 105
})
```

改为依赖按需载入的使用方式：

```js
import('qrcode').then(QRcode => {
  QRcode.toDataURL('xxxx', {
    margin: 2,
    width: 105
  })
})
```

这时候你再通过 vue-cli 进行项目构建时，你会惊喜的发现，又多了一（几）个独立依赖包，基础依赖包的体积也降了下来~

## 总结

通过以上三种方式，我们可以很大程度的减少首屏需要加载的核心文件大小，从而提高页面的加载速度，减少白屏时间。当然白屏时间优化，不止这一种方案，还有很多其他的方式，比如 SSR、Landing Page、静态资源上传 CDN 等，这里就不再一一展开。你可以根据实际的业务场景进一步做优化。

需要注意一点，页面的分包数量并不是越多越好，分包越多，网络请求就越多，在不同平台上网络请求的系统带来的开销可能会比文件体积大小导致的白屏问题更严重。webpack 的动态导入特性的使用需要自己权衡。

-----

### 参考文章

- [前端性能优化之白屏时间](https://juejin.im/post/5d80bd58e51d4561c94b1076)
